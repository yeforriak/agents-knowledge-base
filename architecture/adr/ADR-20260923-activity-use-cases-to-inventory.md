# ADR-20260923: Activity use cases flow to inventory

## Status

Accepted (2026-09-23). Implemented in geordie-monorepo PR #5421 (table and route), prism PR #748 (push), geordie-monorepo PR #5422 (reporting_facts column and `group=useCase`), and geordie-front-end PR #2571 (Cost Intelligence grouping and drilldown).

## Context

Prism now classifies each agent activity into a use case as its trace lands (prism PRs #732 to #734): a one-line description per activity, a per-org taxonomy that a person promotes, and a verdict per activity and taxonomy version. All of this is stored on S3 under the prism data-products bucket.

The product needs to aggregate those verdicts by dimensions prism does not have: a user's tokens by use case over a window, a department's share of each use case, a platform's mix. Inventory already holds one row per activity in `reporting.reporting_facts` with `activity_id`, `user_subject`, `platform`, `model`, `prompt_tokens` and `completion_tokens`. A per-activity use case label joined on `activity_id` turns each of those questions into a `GROUP BY`.

The first attempt (prism PR #735, closed) aggregated a window summary in prism and wrote it to S3. That reproduces inventory's dimensions in the wrong place and cannot answer per-user or per-department questions.

### Affected repositories

- `prism` — push per-activity classifications and taxonomy versions to inventory over the existing `InventoryClient`.
- `geordie-monorepo/agent-inventory` — two new tables, one ingest route, repository, tests.

### Constraints

1. **Tenant isolation.** Every new table carries `org_public_id`; every read and write carries an org predicate. Machine callers have no `x-org-id`; the org is derived server-side, following the precedent of `/intelligence-annotations` and `/agents/{id}/behaviour`.
2. **Realtime and batch.** Verdicts arrive one at a time from the trace consumer and in batches of hundreds from `reclassify_window`. One endpoint accepts a list either way.
3. **Versioned taxonomies.** Promoting a new taxonomy must not overwrite history. Prism keeps every version and every verdict on S3. Inventory needs only the current label per activity, because reporting groups by "what is this activity's use case today".
4. **`reporting_facts` is the read path.** Cost Intelligence groups, filters and drills down through `reporting.reporting_facts`. A dimension that is not a column of that view cannot be a tile grouping or a drilldown filter, so the label is projected into the view at refresh time rather than joined at query time.

---

## Decision

Prism remains the source of truth for descriptions and verdicts on S3, and pushes the product-facing subset to inventory over HTTP with the full payload, no S3 pointers. Inventory stores one row per activity in `inventory` and projects the label into `reporting_facts`, where every Cost Intelligence query already runs.

### Inventory

Migration `V392` adds two tables to the `inventory` schema, next to the other prism-derived activity attributes (annotations, behaviour data); with no foreign key they can move to a schema of their own when the area grows its own aggregate. `inventory.activity_use_cases` holds one row per label, keyed `(org_public_id, activity_id, use_case)`, so the label is the row's identity and later per-label attributes (a primary flag, a confidence, a weight for splitting spend, a `source` for human overrides) are single nullable columns. Confidence is not stored: nothing reads it, and prism keeps it on the S3 verdict. Prism assigns one label per activity today and reporting joins it as such; how one label is picked when the classifier goes multi-label (a primary flag or a score) is decided then. The classifier assigns one label today; the key and the API payload (`labels: [{useCase, proposedLabel}]`) already admit more. `use_case` is bounded to 200 characters because it sits in a btree key. `inventory.activity_use_case_history` is append-only with an identity id, one row per label per push, written in the same transaction, so a label change can be traced. A push replaces the activity's label set. Both tables carry `org_public_id`, so offboarding purges them as Group A.

Promoted taxonomies are mirrored too, since the 2026-09-25 addendum below; proposals that were never promoted stay on S3.

Route, machine callers only (`MACHINE_USERS_REQUIRED`): `POST /activity-use-cases`, body `{scannedByUserExternalId, activityUseCases: [{activityId, labels: [{useCase, confidence, proposedLabel}], description, taxonomyVersion, classifiedAt}]}`. The org is resolved from the `scannedBy` user, exactly as `/intelligence-annotations` does, and never read from the body. Batches above 1,000 rows return 400.

Migration `V393` restates `reporting.reporting_facts` with a `use_case` column: `LEFT JOIN inventory.activity_use_cases` on `(org_public_id, activity_id)` in the activity branch, `NULL::text` elsewhere. `GROUP_BY.USE_CASE` and a `useCases` filter follow the pattern of every other dimension (`group=useCase`, `useCase=...`). An activity without a verdict reports under the existing unresolved group.

The same rebuild drops `scenario_explanation` and `mitigation_suggestions` from the risk covering index. Those two wide columns sat at the btree row-size limit and were read by one detail query; dropping them costs that query a heap fetch per row and removes the risk that a long explanation breaks the refresh.

### Prism

- `ClassifiedActivity`: a verdict joined to its description and a classification time.
- `InventoryClient.push_activity_use_cases(scan_by_user_id, rows)`.
- `classify_and_store` groups the verdicts it wrote by `scan_by_user_id` and pushes one request per group. The realtime path sends one verdict; `reclassify_window` chunks send up to 40. Gated on `settings.inventory_api`, like the behaviour push, so environments without inventory keep working. A failed push raises, so the chunk retries and rewrites the same verdicts.

### Front end

`useCase` joins the spend groupings ("By use case") and the drilldown shows a "Use cases" breakdown beside "Models" for every rung except a use-case rung, which drills into owners.

### Payload versus pointer

Full payload. Inventory has no code path that reads a prism bucket, and neither the annotations push nor the behaviour push uses pointers. A row is a few hundred bytes. A pointer would add cross-service bucket permissions and a second fetch for no saving.

---

## Alternatives Considered

### Option 1: Aggregate in prism, publish summaries to S3 (PR #735)

- Pros: no inventory change, one JSON per window.
- Cons: cannot join on users, departments or tokens; duplicates inventory's dimensions; every new question is a prism release.

### Option 2: Inventory reads verdicts from the prism bucket on an S3 event

- Pros: no HTTP payloads.
- Cons: inventory gains a dependency on a prism bucket layout and IAM; a new consumer to run; inverts the direction every other prism output already uses.

### Option 3: Push over HTTP with full payloads (chosen)

- Pros: same pattern as annotations and behaviour; org derived server-side; realtime and batch through one route; inventory owns the queries.
- Cons: `reclassify_window` over a month is thousands of rows, so batching is required and a failed batch must be retried by the actor.

### Option 4: Join `activity_use_cases` at query time instead of projecting into `reporting_facts`

- Pros: no view rebuild.
- Cons: Cost Intelligence groups and filters only through `reporting_facts` columns, so every grouping, filter and drilldown would need a bespoke join path; the covering indexes would no longer cover. Rejected once the read path was traced.

---

## Consequences

### Positive

- "Tokens by use case for this user this month" is one query in inventory.
- Taxonomy versions and verdicts stay history-preserving on S3; inventory holds the current label only.
- Prism's S3 store stays authoritative and re-pushable.

### Negative

- Two stores to keep consistent. Mitigated by idempotent upserts and by re-running `reclassify_window` to re-push.
- The description text (15 words, model-written) and the proposed label are stored in inventory; the confidence is not. Accepted for the UI's examples; both can be dropped later.
- A label changes only when the view refreshes, so a reclassification shows in Cost Intelligence after the next `refreshIfStale`, not immediately.
- The activities list endpoint has no use-case filter, so drilling from a use case down to an agent's activities lists them unscoped. Follow-up.

---

## Implementation Plan

1. Inventory: `V392`, repository with a two-tenant colliding-id test (goes red when the org leaves the key), route tests, `Server` wiring. Monorepo PR #5421.
2. Prism: client method and actor push with tests. Prism PR #748.
3. Inventory: `V393` rebuild, `GROUP_BY.USE_CASE`, filter and parsing, with a two-tenant test on the view join (goes red when the org pin leaves the join). Stacked on PR #5421.
4. Front end: grouping and drilldown panel.
5. Roll out: merge 1, then 2, then 3 and 4; prism is already on in prod-eu for the Geordie org, so rows land as soon as 2 deploys, and the view picks them up when 3 deploys.

---

## Architecture Impact

- Boundaries: unchanged. Prism produces intelligence and pushes it; inventory stores and serves.
- New abstractions: one repository and one routes class in inventory; two client methods in prism.
- Duplication: none. Discovery, classification and prompts stay in prism; aggregation stays in inventory.

---

## Open Questions

- Whether a new org should start from a shared default taxonomy or wait for its own promoted version. Deferred; today no taxonomy means no classification.
- When multi-label classification arrives, whether spend should be split across labels or stay on the primary. Today the view counts the primary only.
- An endpoint to wipe an org's use cases (admin route plus a `use-cases:reset` machine scope) is a follow-up ticket.

---

## Approval Criteria

This ADR is valid if:

- It respects repository boundaries
- It follows conventions.md
- It aligns with architecture/overview.md

---

## Addendum 2026-09-25: promoted taxonomies are mirrored into inventory

The original decision kept taxonomies out of inventory. Two needs reversed it:

1. The product must know for certain whether an org has a taxonomy. Inferring it from the labels in the selected period hides the use-case surfaces for a window before labelling began and cannot say "taxonomy set, nothing classified yet".
2. A later UI lists an org's taxonomy, its labels with descriptions, and how it changed across versions. `activity_use_cases.taxonomy_version` already names a version that nothing in inventory could resolve.

Prism stays the source of truth and is the only writer. Inventory holds a mirror of promoted taxonomies, pushed by `promote_taxonomy` through a retrying actor, the same direction as every other prism output. Proposals that were never promoted stay on S3.

Migration `V396` adds `inventory.use_case_taxonomies` keyed `(org_public_id, version)` with `source`, `proposal_id`, `supersedes_version`, `promoted_at`, `recorded_at`, and `inventory.use_case_taxonomy_labels` keyed `(org_public_id, version, name)` with `description`, `department_hint`, `position`. The current taxonomy is the newest `promoted_at` per org. Versions are prism's content hash, so a re-push restates a version rather than duplicating it. `name` is bounded to 200 like `activity_use_cases.use_case` so the two join by name.

Route `PUT /use-case-taxonomies` is machine-only and goes through the org authz layer. A promote has no user behind it, so prism names the org in `x-org-id` and is authorised as an internal service: `InternalServiceRepository` registers prism's Auth0 client id from `INTERNAL_SERVICE_PRISM_CLIENT_ID` holding only `USE_CASES_WRITE`, one permission for everything prism writes about an org's use cases (taxonomy pushes, label resets), the same mechanism the scanners client uses for `SCANS_WRITE`. An Auth0 scope was considered and rejected so that machine callers stay inside the authz layer, with one permission enum, one policy and a grant checked in the repository, and can be audited there later. The route still checks the org exists and is active. Org-scoped reads through the grant: `GET /use-case-taxonomies/current` (204 when none), `GET /use-case-taxonomies`, `GET /use-case-taxonomies/{version}`. The front end gates its use-case surfaces on the current taxonomy through a BFF route that degrades to `null`.

Implemented in geordie-monorepo (inventory tables, routes, Auth0 scope), prism PR #760 (push on promote, `sync` tooling) and geordie-front-end PR #2571 (gate).
