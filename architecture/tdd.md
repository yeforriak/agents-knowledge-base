# Test-Driven Development (TDD) for Architecture Integrity

This document defines how and why Test-Driven Development (TDD) must be used when implementing features across repositories.

TDD is not only for correctness — it is a tool to enforce and protect architectural boundaries.

---

# 1. Core Principle

TDD MUST be used to:

* Drive design decisions
* Enforce repository boundaries
* Prevent unintended coupling
* Ensure long-term maintainability

---

# 2. Why TDD Matters for Architecture

In a multi-repository system:

* Architecture can degrade silently
* Logic can leak across boundaries
* Responsibilities can become unclear

TDD prevents this by:

* Forcing you to define behavior before implementation
* Encouraging dependency clarity
* Making violations visible through failing tests

---

# 3. TDD Workflow (MANDATORY)

For every change, follow this sequence:

## Step 1 — Write a Failing Test

* Define the expected behavior first
* Place the test in the correct repository
* Ensure the test reflects architectural intent

## Step 2 — Run and Confirm Failure

* The test MUST fail initially
* If it passes, the test is invalid

## Step 3 — Implement Minimal Code

* Add only what is needed to pass the test
* Do not over-engineer

## Step 4 — Refactor Safely

* Improve structure without changing behavior
* Keep all tests passing

---

# 4. Architectural Rules Enforced by TDD

## 4.1 Respect Repository Responsibilities

---

## 4.2 Test Through Public Interfaces

* Interact only with exposed APIs/modules
* Do not rely on internal implementation details

This ensures:

* Loose coupling
* Replaceable components

---

## 4.3 Prevent Logic Leakage

If a test requires duplicating logic from another repo:

→ STOP

Instead:

* Call the appropriate dependency
* Adjust interfaces if needed

---

# 5. Cross-Repository Development with TDD

When a feature spans multiple repositories

- Step 1 — Start from the Entry Point
- Step 2 — Write High-Level Test, Define expected behavior at system boundary
- Step 3 — Identify Missing Capabilities
- Step 4 — Move to Lower-Level Repo
- Step 5 — Return to Entry Point


---

# 6. Test Design Guidelines

## 6.1 Behavior Over Implementation

* Test WHAT the system does
* Not HOW it does it

## 6.2 Keep Tests Focused

* One behavior per test
* Avoid large, multi-purpose tests

## 6.3 Use Realistic Scenarios

* Reflect actual usage patterns
* Avoid artificial setups

---

# 7. Anti-Patterns (FORBIDDEN)

## ❌ Writing Implementation First

* Leads to poor design decisions

## ❌ Testing Internal Functions

* Creates tight coupling

## ❌ Skipping the Failing Step

* Breaks the TDD cycle

## ❌ Over-Mocking

* Hides architectural problems

## ❌ Duplicating Logic in Tests

* Masks boundary violations

---

# 8. Heuristics for Good Design

If TDD feels difficult, it is a signal:

* Tight coupling → refactor dependencies
* Unclear responsibilities → revisit architecture
* Hard-to-test code → simplify design

TDD friction is feedback, not a problem.

---

# 9. Minimal Implementation Rule

When making tests pass:

* Implement the simplest possible solution
* Avoid premature abstraction
* Generalize only after multiple use cases emerge

---

# 10. Refactoring Rules

Refactoring is allowed ONLY if:

* All tests pass
* Behavior remains unchanged
* Architecture is improved or clarified

---

# 11. Documentation Through Tests

Tests serve as:

* Living documentation
* Examples of usage
* Validation of architecture

Well-written tests should make system behavior obvious.

---

# 12. Agent Execution Strategy

When performing a task:

1. Read architecture documents
2. Identify affected repositories
3. Start with a failing test
4. Implement minimal solution
5. Ensure cross-repo compatibility
6. Refactor safely

---

# 13. Summary

TDD is REQUIRED because it:

* Enforces architecture
* Prevents boundary violations
* Improves design quality
* Reduces regressions

Follow the cycle strictly:

→ Test → Fail → Implement → Refactor

Do not skip steps.
