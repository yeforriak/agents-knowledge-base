# Best Simple System for Now (BSSN)

> Source: Daniel Terhorst-North, "Best Simple System for Now"

The **simplest system** that meets the needs of the product **right now**, written to an **appropriate standard**. No extraneous or over-engineered code; any code it has is exactly as robust as it needs to be, neither more nor less.

## The Three Words

### "For Now"

The system should not anticipate the future.

- No speculative interfaces with single implementations
- No scaling for users you don't have
- No hooks and extension points "just in case"
- No rules engines when you have 3 rules

Simple is a function of now. When requirements change, the definition of simple changes with them.

### "Simple"

Can you remove anything and still meet current requirements? If yes, it doesn't belong.

Flexibility comes from simplicity — not by catering for all futures, but by being so simple it can flex in any direction.

**Gall's Law**: A complex system that works is invariably found to have evolved from a simple system that worked. A complex system designed from scratch never works.

### "Best"

Simple does not mean cutting corners. Quality is appropriate to context.

**Sketching vs Hacking**:
- *Sketching*: Just enough quality to sustain direction; can fill in later or discard with minimal loss
- *Hacking*: Quickly becomes unmanageable

## CUPID Properties

| Property | Meaning |
|----------|---------|
| **C**omposable | Plays nice with other code, minimal dependencies |
| **U**nix philosophy | Does one thing, obviously and comprehensively |
| **P**redictable | Behaviour, observability, failure modes, change impact |
| **I**diomatic | Familiar patterns for the technology |
| **D**omain-based | Intention-revealing naming and structure |

## Decision Checklist

Before adding code:
- Does this solve a *current* requirement?
- Can I remove anything and still meet current needs?
- Is this the appropriate quality level for this context?
- Am I solving for the general case when I should solve for the specific?

## Red Flags: Over-Engineering

- Interfaces with single implementations "for flexibility"
- Generic solutions for specific problems
- Scaling for load you don't have
- DRYing code that happens to look similar but isn't conceptually the same
- Building for hypothetical future requirements

## Red Flags: Under-Engineering

- Sketches that should have been stabilised
- Missing domain language
- Code that's hard to reason about
- Inconsistent patterns
- Copy-paste instead of thoughtful structure

## Build vs Library

```
library_cost = adoption + learning_curve + quirk_workarounds +
               transitive_deps + limitation_workarounds + rip_out_cost

custom_cost  = lines_of_code + testing + maintenance
```

If custom_cost < library_cost, build it. And: is the library the best solution *for now*, not for imagined future needs?

## Key Maxim

> Value costs dwarf effort costs. Optimise for delay and opportunity, not wages.
