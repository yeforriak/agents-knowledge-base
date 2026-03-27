# Agent Workflow Discipline

Process rules for disciplined implementation. These prevent the common failure mode where code changes are made but the surrounding work (docs, verification, communication) is missed.

## Define done before starting

Before implementing, explicitly state what "done" looks like — acceptance criteria and how to verify them. This prevents half-finished work where the code change lands but docs, tests, or other artefacts are missed.

Ask: "What needs to be true for this to be complete?" Then work backwards from that.

## Distinguish thinking from instruction

When the user's language suggests exploration ("I'm thinking about...", "Can we...", "What if...") rather than direction ("Do X", "Add Y"), verify the plan of action and get confirmation before implementing. Treat exploratory language as a prompt for discussion, not a directive.

Exception: if the user has explicitly said to proceed in advance.

## Keep docs consistent after changes

After making code or behaviour changes, check that all relevant documentation remains consistent. This includes README, changelog, architecture docs, contributing guide, and any other docs that reference the changed behaviour.

Docs drifting from reality is a bug.

## Prompt to commit after meaningful work

After completing a meaningful chunk of work (feature, fix, refactor), proactively ask the user if they want to commit and propose a commit message using Angular/semantic conventions. Don't commit without asking, but don't wait to be told either.

## Verify functional state after changes

Always verify the system is in a functional state after making changes — tests pass, linter clean, build succeeds. Do this before declaring work complete or asking about committing.

A broken state is never "done".
