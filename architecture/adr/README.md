# ADR Process for AI Agents

All non-trivial changes MUST begin with an ADR.

---

# 1. When an ADR is REQUIRED

An ADR must be created BEFORE implementation if:

* Multiple repositories are involved
* New functionality is introduced
* Existing behavior is modified
* Architecture decisions are required

---

# 2. Workflow (MANDATORY)

## Step 1 — Create ADR

* Use template.md
* Place file in architecture/adr/
* Use filename: ADR-<timestamp>-<short-title>.md

## Step 2 — Think Before Coding

* Analyze the problem
* Identify affected repositories
* Evaluate alternatives

## Step 3 — Define Architecture

* Specify responsibilities per repo
* Ensure no duplication of logic

## Step 4 — Only Then Implement

* Follow TDD (see tdd.md)
* Follow conventions.md

---

# 3. Constraints

* Do NOT skip ADR step
* Do NOT implement before writing ADR
* Do NOT create vague ADRs

---

# 4. Quality Requirements

A good ADR:

* Clearly defines the problem
* Explains WHY the decision is made
* Mentions trade-offs
* Respects architecture boundaries

---

# 5. Output Expectations (for AI agents)

Before writing code, you MUST:

1. Produce a complete ADR
2. Ensure it is actionable
3. Ensure it includes implementation steps

Only after that:
→ proceed with coding

---

# 6. Failure Conditions

The ADR is INVALID if:

* It does not specify affected repositories
* It duplicates logic across repos
* It lacks a clear implementation plan
* It violates conventions.md

---

# 7. Summary

No ADR → No Implementation

This rule is strict.
