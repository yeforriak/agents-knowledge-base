# Engineering Conventions for AI Agents

This document defines the rules and expectations for modifying code across repositories.
All changes MUST follow these conventions unless explicitly instructed otherwise.

---

# 1. General Principles

## 1.1 Consistency Over Perfection

* Follow existing patterns in the repository
* Do not introduce new abstractions unless necessary
* Prefer alignment with current code over theoretical improvements

## 1.2 Minimal Changes

* Make the smallest change that solves the problem
* Avoid refactoring unrelated code
* Do not rename files, functions, or variables unless required

## 1.3 Readability First

* Code must be easy to understand for humans
* Avoid clever or overly compact solutions
* Prefer explicit logic over implicit behavior

---

# 2. Repository Awareness

## 2.1 Respect Boundaries

Each repository has a clear responsibility:

* **repoA** → domain / business logic
* **repoC** → API / orchestration layer

Rules:

* Do NOT duplicate business logic across repositories
* repoC may call repoA, but not reimplement it
* Cross-repo changes must remain compatible

---

## 2.2 Cross-Repository Changes

When modifying multiple repositories:

* Ensure interfaces remain consistent
* Update all affected call sites
* Document assumptions in code comments
* Prefer backward-compatible changes

---

# 3. Code Style

## 3.1 Follow Existing Style

* Match formatting, naming, and structure already used
* Do not introduce new linting rules or formatters
* Do not reformat entire files

---

## 3.2 Naming Conventions

* Use descriptive names
* Avoid abbreviations unless already used
* Functions → verbs (`getUser`, `createOrder`)
* Variables → nouns (`user`, `orderList`)

---

## 3.3 File Organization

* Keep related logic together
* Do not create new top-level folders without strong reason
* Prefer extending existing modules

---

# 4. Dependencies

## 4.1 Adding Dependencies

* Avoid adding new dependencies unless necessary
* Prefer built-in or existing libraries
* Justify any new dependency in comments

---

## 4.2 Versioning

* Do not upgrade dependencies unless required
* Do not introduce breaking changes

---

# 5. Testing

## 5.1 Test Requirements

* Add tests for new functionality
* Update tests when behavior changes
* Do not remove tests unless invalid

---

## 5.2 Scope

* Test the behavior, not implementation details
* Cover edge cases when relevant

---

# 6. Error Handling

* Do not swallow errors silently
* Provide meaningful error messages
* Maintain existing error-handling patterns

---

# 7. Logging

* Use existing logging utilities
* Do not introduce new logging frameworks
* Avoid excessive logging

---

# 8. Documentation

## 8.1 When to Document

* Add comments for non-obvious logic
* Document public APIs
* Explain cross-repo interactions

---

## 8.2 What NOT to Document

* Do not restate obvious code
* Avoid redundant comments

---

# 9. Git Conventions

## 9.1 Commits

* Keep commits small and focused
* One logical change per commit

## 9.2 Commit Messages

Format: <type>: <short description>

Examples:

* feat: add user validation
* fix: handle null response in API

---

# 10. Forbidden Actions

Unless explicitly instructed, DO NOT:

* Rewrite large parts of the codebase
* Introduce new architectures or frameworks
* Rename core modules
* Modify unrelated files
* Add global state
* Bypass existing abstractions

---

# 11. Preferred Workflow for Tasks

When implementing a feature:

1. Understand context (read architecture docs)
2. Identify affected repositories
3. Locate existing patterns
4. Implement minimal solution
5. Update tests
6. Verify cross-repo compatibility

---

# 12. Decision Making Heuristics

When unsure:

1. Choose the simplest solution
2. Follow existing patterns
3. Avoid introducing new concepts
4. Prefer explicit over implicit
5. Minimize surface area of change

---

# 13. Communication (for AI agents)

When making changes:

* Explain WHY, not just WHAT
* Highlight cross-repo impact
* Mention assumptions
* Call out trade-offs

---

# 14. Priority Order

If rules conflict, follow this priority:

1. Explicit task instructions
2. Existing code patterns
3. This document
4. General best practices

---

# 15. Summary

* Be conservative
* Be consistent
* Be minimal
* Respect boundaries
* Keep code understandable

This is more important than writing "perfect" code.
