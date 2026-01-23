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
| [Technology Reference](System%20Design%20Interview%20Patterns.md#technology-reference-by-layer) | Tools organized by architectural layer |

### Root Insurance Interview Questions

| Question | Key Topics |
|----------|------------|
| [Telematics Data Pipeline](System%20Design%20Interview%20Patterns.md#question-1-design-a-telematics-data-pipeline) | Kafka, Flink, mobile batching, feature extraction |
| [Fraud Detection System](System%20Design%20Interview%20Patterns.md#question-2-design-a-real-time-fraud-detection-system) | Rules engine, ML anomaly detection, graph analysis |
| [Pricing Engine](System%20Design%20Interview%20Patterns.md#question-3-design-a-usage-based-insurance-pricing-engine) | Quote orchestration, caching, regulatory compliance |
| [Location Tracking](System%20Design%20Interview%20Patterns.md#question-4-design-a-real-time-location-tracking-system) | WebSockets, Redis geospatial, matching |
| [Analytics Dashboard](System%20Design%20Interview%20Patterns.md#question-5-design-a-real-time-analytics-dashboard) | Druid, ClickHouse, pre-aggregation |

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
