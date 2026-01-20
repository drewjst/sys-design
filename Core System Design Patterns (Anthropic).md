# Core System Design Patterns

A reference guide for the fundamental architectural patterns that solve most system design problems.

---

## 1. Read-Heavy Pattern (Cache-Aside + Read Replicas)

**Use when:** Read:Write ratio > 10:1  
**Examples:** Social media feeds, product catalogs, content sites, user profiles

```mermaid
flowchart TB
    subgraph Clients
        C1[Client]
        C2[Client]
    end

    subgraph "Load Balancing"
        LB[Load Balancer]
    end

    subgraph "API Layer"
        API1[API Server]
        API2[API Server]
    end

    subgraph "Caching Layer"
        CACHE[(Redis/Memcached)]
    end

    subgraph "Database Layer"
        PRIMARY[(Primary DB)]
        REPLICA1[(Read Replica)]
        REPLICA2[(Read Replica)]
    end

    subgraph "CDN"
        CDN1[CDN Edge]
    end

    C1 --> CDN1
    C2 --> CDN1
    CDN1 --> LB
    LB --> API1
    LB --> API2
    
    API1 --> |1. Check Cache| CACHE
    API2 --> |1. Check Cache| CACHE
    
    CACHE -.-> |2. Cache Miss| REPLICA1
    CACHE -.-> |2. Cache Miss| REPLICA2
    
    API1 --> |Writes| PRIMARY
    API2 --> |Writes| PRIMARY
    
    PRIMARY --> |Async Replication| REPLICA1
    PRIMARY --> |Async Replication| REPLICA2
```

**Key decisions:**
- Cache invalidation strategy (TTL vs event-based)
- Replication lag tolerance
- Cache warming on cold start

---

## 2. Write-Heavy Pattern (Buffered Writes + Async Processing)

**Use when:** High write throughput, eventual consistency acceptable  
**Examples:** Analytics, logging, IoT telemetry, metrics collection

```mermaid
flowchart TB
    subgraph Clients
        C1[Client]
        C2[Client]
        C3[Client]
    end

    subgraph "Ingestion Layer"
        LB[Load Balancer]
        ING1[Ingestion Service]
        ING2[Ingestion Service]
    end

    subgraph "Buffer Layer"
        KAFKA[Kafka / Kinesis]
    end

    subgraph "Processing Layer"
        PROC1[Stream Processor]
        PROC2[Stream Processor]
        PROC3[Stream Processor]
    end

    subgraph "Storage Layer"
        TS[(Time-Series DB)]
        DW[(Data Warehouse)]
        BLOB[(Object Storage)]
    end

    C1 --> LB
    C2 --> LB
    C3 --> LB
    
    LB --> ING1
    LB --> ING2
    
    ING1 --> |Async Write| KAFKA
    ING2 --> |Async Write| KAFKA
    
    KAFKA --> PROC1
    KAFKA --> PROC2
    KAFKA --> PROC3
    
    PROC1 --> TS
    PROC2 --> DW
    PROC3 --> BLOB
```

**Key decisions:**
- Partition strategy for parallelism
- At-least-once vs exactly-once semantics
- Batch size and flush intervals
- Dead letter queue for failures

---

## 3. Real-Time Bidirectional Pattern (WebSocket + Pub/Sub)

**Use when:** Low-latency, bidirectional communication required  
**Examples:** Chat applications, collaborative editing, live gaming, trading platforms

```mermaid
flowchart TB
    subgraph Clients
        C1[Client]
        C2[Client]
        C3[Client]
    end

    subgraph "Connection Layer"
        LB[Load Balancer<br/>Sticky Sessions]
        WS1[WebSocket Server]
        WS2[WebSocket Server]
    end

    subgraph "Pub/Sub Layer"
        REDIS[(Redis Pub/Sub)]
    end

    subgraph "State Management"
        PRESENCE[Presence Service]
        CONN_REG[(Connection Registry)]
    end

    subgraph "Persistence"
        DB[(Database)]
        CACHE[(Message Cache)]
    end

    C1 <--> |WebSocket| LB
    C2 <--> |WebSocket| LB
    C3 <--> |WebSocket| LB
    
    LB <--> WS1
    LB <--> WS2
    
    WS1 <--> |Subscribe/Publish| REDIS
    WS2 <--> |Subscribe/Publish| REDIS
    
    WS1 --> PRESENCE
    WS2 --> PRESENCE
    PRESENCE --> CONN_REG
    
    WS1 --> |Persist Messages| DB
    WS1 --> |Recent Messages| CACHE
```

**Key decisions:**
- Connection state management across servers
- Heartbeat/keepalive strategy
- Message ordering guarantees
- Reconnection and message replay

---

## 4. Event-Driven / Choreography Pattern

**Use when:** Loose coupling between services, complex workflows  
**Examples:** Order processing, payment flows, inventory management, notifications

```mermaid
flowchart TB
    subgraph "Entry Point"
        API[API Gateway]
    end

    subgraph "Event Backbone"
        EB[Event Bus<br/>Kafka/SNS+SQS]
    end

    subgraph "Domain Services"
        ORDER[Order Service]
        PAYMENT[Payment Service]
        INVENTORY[Inventory Service]
        SHIPPING[Shipping Service]
        NOTIFY[Notification Service]
    end

    subgraph "Service Storage"
        DB1[(Orders DB)]
        DB2[(Payments DB)]
        DB3[(Inventory DB)]
        DB4[(Shipping DB)]
    end

    API --> ORDER
    
    ORDER --> |OrderCreated| EB
    ORDER --> DB1
    
    EB --> |OrderCreated| PAYMENT
    EB --> |OrderCreated| INVENTORY
    
    PAYMENT --> |PaymentProcessed| EB
    PAYMENT --> DB2
    
    INVENTORY --> |InventoryReserved| EB
    INVENTORY --> DB3
    
    EB --> |PaymentProcessed<br/>InventoryReserved| SHIPPING
    
    SHIPPING --> |ShipmentCreated| EB
    SHIPPING --> DB4
    
    EB --> |All Events| NOTIFY
```

**Key decisions:**
- Event schema evolution strategy
- Saga pattern for distributed transactions
- Idempotency for event handlers
- Dead letter handling and compensation

---

## 5. CQRS Pattern (Command Query Responsibility Segregation)

**Use when:** Read and write models have different shapes or scale requirements  
**Examples:** E-commerce search, reporting dashboards, complex domain models

```mermaid
flowchart TB
    subgraph Clients
        C1[Client - Reads]
        C2[Client - Writes]
    end

    subgraph "API Layer"
        QUERY_API[Query API]
        CMD_API[Command API]
    end

    subgraph "Command Side"
        CMD_SVC[Command Service]
        WRITE_DB[(Write DB<br/>Normalized)]
    end

    subgraph "Event Stream"
        EVENTS[Event Store / Stream]
    end

    subgraph "Projection Services"
        PROJ1[Projector - Search]
        PROJ2[Projector - Analytics]
    end

    subgraph "Query Side"
        QUERY_SVC[Query Service]
        ES[(Elasticsearch)]
        CACHE[(Redis Cache)]
        ANALYTICS[(Analytics DB)]
    end

    C1 --> QUERY_API
    C2 --> CMD_API
    
    CMD_API --> CMD_SVC
    CMD_SVC --> WRITE_DB
    CMD_SVC --> |Emit Events| EVENTS
    
    EVENTS --> PROJ1
    EVENTS --> PROJ2
    
    PROJ1 --> ES
    PROJ1 --> CACHE
    PROJ2 --> ANALYTICS
    
    QUERY_API --> QUERY_SVC
    QUERY_SVC --> ES
    QUERY_SVC --> CACHE
    QUERY_SVC --> ANALYTICS
```

**Key decisions:**
- Eventual consistency window tolerance
- Projection rebuild strategy
- Event versioning
- Read model technology per use case

---

## Quick Selection Matrix

| Pattern | Read:Write Ratio | Consistency | Latency | Complexity |
|---------|-----------------|-------------|---------|------------|
| Read-Heavy | High reads | Strong possible | Low reads | Low |
| Write-Heavy | High writes | Eventual | Async writes | Medium |
| Real-Time | Balanced | Eventual | Very low | High |
| Event-Driven | Varies | Eventual | Async | High |
| CQRS | High reads, complex writes | Eventual | Optimized per side | Very High |

---

## Combining Patterns

Most real systems combine these patterns:

```mermaid
flowchart TB
    subgraph "User-Facing"
        direction TB
        A[Read-Heavy Pattern]
    end
    
    subgraph "Background Processing"
        direction TB
        B[Write-Heavy Pattern]
    end
    
    subgraph "Real-Time Features"
        direction TB
        C[WebSocket Pattern]
    end
    
    subgraph "Service Integration"
        direction TB
        D[Event-Driven Pattern]
    end
    
    A <--> D
    B <--> D
    C <--> D
```

**Example: Twitter-like System**
- Read-Heavy: Timeline rendering with fan-out on read
- Write-Heavy: Analytics and engagement metrics
- Real-Time: Notifications and live updates
- Event-Driven: Cross-service coordination (tweets → notifications → analytics)

---

## Interview Tips

1. **Start with requirements** - Clarify read/write ratio, latency needs, consistency requirements
2. **Name the pattern** - "This is a classic read-heavy workload, so I'll use cache-aside with read replicas"
3. **Draw the boxes first** - Get the high-level architecture before diving into specifics
4. **Justify tool choices** - "I'd use Kafka here because we need replay capability and high throughput"
5. **Discuss tradeoffs** - Every choice has costs; show you understand them