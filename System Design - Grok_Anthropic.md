# System Design Patterns: Interview Reference

There are only a handful of fundamental architectural patterns that solve most system design problems. This repo distills them into quick-reference diagrams you can internalize before interviews.

## Pattern Selection Matrix

| Pattern | When to Use | Read:Write | Consistency | Latency |
|---------|-------------|------------|-------------|---------|
| [Read-Heavy](#1-read-heavy-pattern) | Reads dominate (>10:1) | High reads | Strong possible | Low reads |
| [Write-Heavy](#2-write-heavy-pattern) | High ingestion, eventual OK | High writes | Eventual | Async writes |
| [Real-Time](#3-real-time-bidirectional-pattern) | Live updates required | Balanced | Eventual | Very low |
| [Event-Driven](#4-event-driven-pattern) | Service coordination | Varies | Eventual | Async |
| [CQRS](#5-cqrs-pattern) | Read/write models diverge | Complex | Eventual | Optimized |

**Interview tip:** Start every answer by identifying which pattern fits, then justify your choice.

---

## 1. Read-Heavy Pattern

**The shape:** Cache-aside + read replicas  
**Use when:** Social feeds, product catalogs, user profiles, content sites  
**Read:Write ratio:** >10:1

```mermaid
flowchart TB
    subgraph Clients
        C[Clients]
    end

    subgraph Edge
        CDN[CDN]
    end

    subgraph "Application Tier"
        LB[Load Balancer]
        API1[API Server]
        API2[API Server]
    end

    subgraph "Caching Layer"
        CACHE[(Redis / Memcached)]
    end

    subgraph "Database Tier"
        PRIMARY[(Primary DB)]
        READ1[(Read Replica)]
        READ2[(Read Replica)]
    end

    C --> CDN
    CDN --> LB
    LB --> API1 & API2

    API1 & API2 --> |"1. Check cache"| CACHE
    CACHE -.-> |"2. Miss → query replica"| READ1 & READ2
    CACHE -.-> |"3. Populate cache"| CACHE

    API1 & API2 --> |"Writes"| PRIMARY
    PRIMARY --> |"Async replication"| READ1 & READ2
```

### Key Decisions to Discuss

| Decision | Options | Tradeoff |
|----------|---------|----------|
| Cache invalidation | TTL vs event-based | Simplicity vs freshness |
| Replication | Sync vs async | Consistency vs write latency |
| Cache warming | Eager vs lazy | Startup time vs complexity |

### What Interviewers Want to Hear

- "I'd use cache-aside because it handles cache failures gracefully—we fall back to the replica"
- "With async replication, we accept ~100ms staleness for 10x read throughput"
- "Cache hit rate target is 95%+; below that, we're just adding latency"

---

## 2. Write-Heavy Pattern

**The shape:** Buffer → async process → optimized storage  
**Use when:** Analytics, logging, IoT telemetry, metrics, event tracking  
**Write:Read ratio:** >5:1 or bursty writes

```mermaid
flowchart TB
    subgraph Clients
        C1[Client]
        C2[Client]
        C3[Client]
    end

    subgraph "Ingestion Tier"
        LB[Load Balancer]
        ING1[Ingestion API]
        ING2[Ingestion API]
    end

    subgraph "Buffer Layer"
        KAFKA[Kafka / Kinesis]
        P1[Partition 1]
        P2[Partition 2]
        P3[Partition N]
    end

    subgraph "Processing Tier"
        PROC[Stream Processors<br/>Flink / Spark]
    end

    subgraph "Storage Tier"
        TS[(Time-Series DB<br/>InfluxDB)]
        DW[(Data Warehouse<br/>BigQuery)]
        BLOB[(Object Storage<br/>S3)]
    end

    C1 & C2 & C3 --> LB
    LB --> ING1 & ING2

    ING1 & ING2 --> |"Async write"| KAFKA
    KAFKA --> P1 & P2 & P3

    P1 & P2 & P3 --> PROC

    PROC --> TS
    PROC --> DW
    PROC --> BLOB
```

### Key Decisions to Discuss

| Decision | Options | Tradeoff |
|----------|---------|----------|
| Delivery guarantee | At-least-once vs exactly-once | Throughput vs complexity |
| Partitioning | Hash vs range vs random | Hot spots vs ordering |
| Batch size | Small vs large | Latency vs throughput |

### What Interviewers Want to Hear

- "Kafka gives us replay capability—if a consumer dies, we reprocess from offset"
- "Partitioning by user_id keeps related events ordered but risks hot partitions for power users"
- "We batch writes to the warehouse in 1-minute windows to reduce costs"

---

## 3. Real-Time Bidirectional Pattern

**The shape:** WebSockets + pub/sub fan-out  
**Use when:** Chat, collaboration, gaming, live dashboards, trading  
**Key requirement:** Sub-second latency, bidirectional

```mermaid
flowchart TB
    subgraph Clients
        C1[Client A]
        C2[Client B]
        C3[Client C]
    end

    subgraph "Connection Tier"
        LB[Load Balancer<br/>Sticky Sessions]
        WS1[WebSocket Server]
        WS2[WebSocket Server]
    end

    subgraph "Coordination"
        PUBSUB[(Redis Pub/Sub)]
        PRESENCE[Presence Service]
    end

    subgraph "Persistence"
        MSG_CACHE[(Recent Messages<br/>Redis)]
        DB[(Message Store<br/>Cassandra)]
    end

    C1 <--> |"WebSocket"| LB
    C2 <--> |"WebSocket"| LB
    C3 <--> |"WebSocket"| LB

    LB <--> WS1 & WS2

    WS1 <--> |"Pub/Sub"| PUBSUB
    WS2 <--> |"Pub/Sub"| PUBSUB

    WS1 & WS2 --> PRESENCE
    WS1 & WS2 --> MSG_CACHE
    MSG_CACHE --> DB
```

### Key Decisions to Discuss

| Decision | Options | Tradeoff |
|----------|---------|----------|
| Connection routing | Sticky vs stateless | Simplicity vs failover |
| Message ordering | Per-channel vs global | Correctness vs performance |
| Offline handling | Queue vs drop | Completeness vs resource use |

### What Interviewers Want to Hear

- "Redis pub/sub lets Server A notify Server B's clients without direct coupling"
- "Sticky sessions simplify state but we need connection draining for deploys"
- "On reconnect, client sends last message ID; we replay from the cache"

---

## 4. Event-Driven Pattern

**The shape:** Services emit events, others react (choreography)  
**Use when:** Order flows, payments, inventory, notifications, microservices  
**Key requirement:** Loose coupling, complex workflows

```mermaid
flowchart TB
    subgraph "Entry"
        API[API Gateway]
    end

    subgraph "Event Backbone"
        BUS[Event Bus<br/>Kafka / SNS+SQS]
    end

    subgraph "Domain Services"
        ORDER[Order Service]
        PAYMENT[Payment Service]
        INVENTORY[Inventory Service]
        SHIPPING[Shipping Service]
        NOTIFY[Notification Service]
    end

    subgraph "Per-Service Storage"
        DB1[(Orders)]
        DB2[(Payments)]
        DB3[(Inventory)]
        DB4[(Shipments)]
    end

    API --> ORDER
    ORDER --> DB1
    ORDER --> |"OrderCreated"| BUS

    BUS --> |"OrderCreated"| PAYMENT
    BUS --> |"OrderCreated"| INVENTORY

    PAYMENT --> DB2
    PAYMENT --> |"PaymentProcessed"| BUS

    INVENTORY --> DB3
    INVENTORY --> |"InventoryReserved"| BUS

    BUS --> |"Both events"| SHIPPING
    SHIPPING --> DB4
    SHIPPING --> |"Shipped"| BUS

    BUS --> |"All events"| NOTIFY
```

### Key Decisions to Discuss

| Decision | Options | Tradeoff |
|----------|---------|----------|
| Coordination | Choreography vs orchestration | Autonomy vs visibility |
| Failures | Saga compensation vs 2PC | Availability vs consistency |
| Idempotency | Event ID dedup | Exactly-once semantics |

### What Interviewers Want to Hear

- "Choreography means no central coordinator—services react independently"
- "For payment failures, we use a saga: emit PaymentFailed, Inventory listens and releases reservation"
- "Every handler checks event ID to ensure idempotent processing"

---

## 5. CQRS Pattern

**The shape:** Separate read and write models  
**Use when:** Search over transactional data, complex reporting, different scaling needs  
**Key requirement:** Read and write shapes diverge significantly

```mermaid
flowchart TB
    subgraph "Clients"
        R[Read Clients]
        W[Write Clients]
    end

    subgraph "Command Side"
        CMD_API[Command API]
        CMD_SVC[Command Handler]
        WRITE_DB[(Write DB<br/>Normalized)]
    end

    subgraph "Event Stream"
        EVENTS[Event Log]
    end

    subgraph "Projections"
        PROJ1[Search Projector]
        PROJ2[Analytics Projector]
    end

    subgraph "Query Side"
        QUERY_API[Query API]
        ES[(Elasticsearch)]
        ANALYTICS[(ClickHouse)]
        CACHE[(Redis)]
    end

    W --> CMD_API --> CMD_SVC --> WRITE_DB
    CMD_SVC --> |"Emit"| EVENTS

    EVENTS --> PROJ1 --> ES
    EVENTS --> PROJ1 --> CACHE
    EVENTS --> PROJ2 --> ANALYTICS

    R --> QUERY_API
    QUERY_API --> ES & CACHE & ANALYTICS
```

### Key Decisions to Discuss

| Decision | Options | Tradeoff |
|----------|---------|----------|
| Consistency window | Seconds vs minutes | UX vs complexity |
| Projection rebuild | Replay all vs snapshot | Recovery time vs storage |
| Event schema | Versioned vs flexible | Evolution vs validation |

### What Interviewers Want to Hear

- "CQRS lets us scale reads and writes independently with different tech"
- "The search index is eventually consistent—typically 2-3 seconds behind"
- "If we need to change the read model, we replay events to rebuild the projection"

---

## Combining Patterns (Real Systems)

Most production systems combine multiple patterns:

```mermaid
flowchart LR
    subgraph "Twitter Example"
        A[Read-Heavy<br/>Timeline Cache]
        B[Write-Heavy<br/>Analytics Pipeline]
        C[Real-Time<br/>Notifications]
        D[Event-Driven<br/>Service Mesh]
    end

    D --> A
    D --> B
    D --> C
```

| System | Primary Pattern | Secondary Patterns |
|--------|-----------------|-------------------|
| Twitter | Read-Heavy (timelines) | Write-Heavy (analytics), Real-Time (notifications) |
| Uber | Real-Time (location) | Event-Driven (ride flow), Write-Heavy (telemetry) |
| Shopify | CQRS (catalog + orders) | Event-Driven (fulfillment), Read-Heavy (storefront) |

---

## Interview Framework

### Step 1: Clarify Requirements (2 min)
- Read vs write ratio?
- Latency requirements?
- Consistency needs?
- Scale (QPS, data size)?

### Step 2: Identify Pattern (1 min)
> "This is a read-heavy workload with a 100:1 read/write ratio, so I'll use cache-aside with read replicas."

### Step 3: Draw High-Level (5 min)
- Boxes first, arrows second
- Name the components generically, then specific tools

### Step 4: Deep Dive (10 min)
- Pick 2-3 components to detail
- Discuss tradeoffs unprompted
- Mention failure modes

### Step 5: Scale & Extend (5 min)
- "At 10x scale, I'd add..."
- "For global users, I'd introduce..."

---

## Quick Reference: Tool Selection

| Need | Options | When to Choose |
|------|---------|----------------|
| Cache | Redis vs Memcached | Redis if you need data structures; Memcached for pure K/V |
| Queue | Kafka vs SQS vs RabbitMQ | Kafka for replay/streaming; SQS for simplicity; Rabbit for routing |
| Search | Elasticsearch vs Algolia | ES for control; Algolia for speed-to-market |
| Time-series | InfluxDB vs TimescaleDB | InfluxDB for metrics; Timescale if you need SQL |
| Write-optimized DB | Cassandra vs ScyllaDB | Scylla for lower latency; Cassandra for ecosystem |

---

## Further Reading

- [System Design Primer](https://github.com/donnemartin/system-design-primer) — Comprehensive GitHub resource
- [ByteByteGo](https://bytebytego.com) — Visual system design guides
- [Hello Interview Patterns](https://www.hellointerview.com/learn/system-design/in-a-hurry/patterns) — Interview-focused patterns
- [DesignGurus.io](https://www.designgurus.io/blog/read-heavy-vs-write-heavy) — Workload optimization guides
