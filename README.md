# System Design Interview Prep

Resources for system design interviews, with a focus on insurtech and data-intensive applications.

## Quick Links

- **[System Design Interview Patterns](System%20Design%20Interview%20Patterns.md)** — Core patterns, technology reference, and interview framework

### Key Sections

| Section | Description |
|---------|-------------|
| [Pattern Selection Matrix](System%20Design%20Interview%20Patterns.md#pattern-selection-matrix) | Quick reference for choosing the right pattern |
| [CAP Theorem](System%20Design%20Interview%20Patterns.md#cap-theorem) | Consistency vs availability tradeoffs |
| [Strangler Fig Pattern](System%20Design%20Interview%20Patterns.md#6-strangler-fig-pattern) | Incremental monolith migration |
| [Root Insurance Questions](System%20Design%20Interview%20Patterns.md#root-insurance-interview-questions) | Telematics, fraud detection, pricing engine |
| [Technology Reference](System%20Design%20Interview%20Patterns.md#technology-reference-by-layer) | Tools organized by architectural layer |

## Patterns Covered

1. **Read-Heavy** — Cache-aside + read replicas
2. **Write-Heavy** — Buffer → async process → optimized storage
3. **Real-Time** — WebSockets + pub/sub fan-out
4. **Event-Driven** — Services emit events, others react
5. **CQRS** — Separate read and write models
6. **Strangler Fig** — Incremental legacy migration

## Interview Framework

```
1. Clarify Requirements (2 min)
2. Identify Pattern (1 min)
3. Draw High-Level (5 min)
4. Deep Dive (10 min)
5. Scale & Extend (5 min)
```
