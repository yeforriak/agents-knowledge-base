# Domain-Driven Design with Ports & Adapters

## Core Principle

The domain model is the centre of the system. Infrastructure (databases, APIs, frameworks, Docker) lives at the edges. Dependencies point inward — the domain never imports from infrastructure.

```
    Adapters (infrastructure)
        ↓
    Ports (interfaces)
        ↓
    Domain (pure logic)
```

## Hexagonal Architecture

The domain defines **ports** — interfaces describing what it needs from the outside world. **Adapters** implement those ports with concrete technology.

```
[CLI Adapter] → [Port: ContainerRuntime] ← [Docker Adapter]
                                          ← [gVisor Adapter]
                                          ← [Sprites Adapter]
```

Swapping Docker for gVisor or Sprites means writing a new adapter. The domain and CLI don't change.

## When to Apply

**Use ports & adapters when:**
- The technology choice is likely to change (container runtime, AI model provider)
- You need to test domain logic without infrastructure (test doubles for the port)
- Multiple adapters for the same port exist or are foreseeable

**Don't use ports & adapters when:**
- There's only one possible implementation and that won't change
- The indirection cost exceeds the benefit (BSSN applies — don't abstract for imagined futures)

## DDD Building Blocks

### Value Objects

Immutable, defined by their attributes, no identity. Compare by value.

```python
# Identity name is a value object — "hawk-alpha" == "hawk-alpha" regardless of which object
class Identity:
    def __eq__(self, other): return self.name == other.name
```

### Entities

Have identity that persists across state changes. Compare by ID, not attributes.

### Aggregates

Cluster of entities/value objects treated as a unit. One entity is the aggregate root — all external access goes through it.

### Repositories

Provide collection-like access to aggregates. The port defines the interface; the adapter provides storage.

### Domain Services

Operations that don't naturally belong to a single entity or value object. Stateless. Named after the operation they perform, not their structural role.

## Naming

Use the domain's language, not technical jargon:
- `Identity`, not `ContainerNameGenerator`
- `foundation`, not `config_repo`
- `ralph_loop`, not `agent_execution_cycle`

Avoid: utility, helper, manager, handler, service (when used generically).

## Bounded Contexts

Different parts of the system may use the same word differently. Define explicit boundaries where a term has one precise meaning. The ubiquitous language document (`docs/ubiquitous-language.md`) is the canonical reference.

## For Razmada

| Concept | Layer | Example |
|---------|-------|---------|
| Identity generation | Domain | `identity.py` — pure logic, no dependencies |
| Container lifecycle | Port + Adapter | Port defines create/list/purge; Docker adapter implements |
| CLI | Adapter | Translates user commands to domain/port calls |
| Foundation cloning | Infrastructure | Adapter concern — how repos get cloned |
| Lore | Domain knowledge | Pure content, no code dependencies |

The domain asks "what" (create a container with this identity for these repos). The adapter answers "how" (Docker, gVisor, Sprites).
