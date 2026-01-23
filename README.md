# System Design Interview Prep

**[Patterns](System%20Design%20Interview%20Patterns.md#pattern-selection-matrix)** · **[CAP Theorem](System%20Design%20Interview%20Patterns.md#cap-theorem)** · **[Strangler Fig](System%20Design%20Interview%20Patterns.md#6-strangler-fig-pattern)** · **[Technology Reference](System%20Design%20Interview%20Patterns.md#technology-reference-by-layer)** · **[Tool Selection](System%20Design%20Interview%20Patterns.md#tool-selection-quick-reference)**

### Root Insurance Questions

[Telematics Pipeline](System%20Design%20Interview%20Patterns.md#question-1-design-a-telematics-data-pipeline) · [Fraud Detection](System%20Design%20Interview%20Patterns.md#question-2-design-a-real-time-fraud-detection-system) · [Pricing Engine](System%20Design%20Interview%20Patterns.md#question-3-design-a-usage-based-insurance-pricing-engine) · [Location Tracking](System%20Design%20Interview%20Patterns.md#question-4-design-a-real-time-location-tracking-system) · [Analytics Dashboard](System%20Design%20Interview%20Patterns.md#question-5-design-a-real-time-analytics-dashboard)

---

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
