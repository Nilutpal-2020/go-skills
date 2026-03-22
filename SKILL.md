# Senior Backend Engineer
---

# API & Backend System Design Patterns (Go-Oriented)

> Focus: **Scalability, correctness, observability, and maintainability**

---

## 1. Resource Design & API Semantics

### Core Principles

* Model APIs around **business resources**, not DB tables
* Prefer **predictable, consistent interfaces**
* Design for **evolution without breaking clients**

---

### REST Patterns (Baseline)

```
GET    /users
POST   /users
GET    /users/{id}
PATCH  /users/{id}
DELETE /users/{id}
```

---

### Advanced Design Decisions (Senior-Level)

#### When to use nested resources

```
GET /users/{id}/orders
```

✔ When relationship is **strong & scoped**

Avoid deep nesting:

```
/users/{id}/orders/{id}/items/{id}
```

Prefer:

```
GET /order-items?order_id=xxx
```

---

#### Actions vs State Transitions

Instead of:

```
POST /orders/{id}/cancel
```

Prefer modeling state:

```
PATCH /orders/{id}
{
  "status": "cancelled"
}
```

Use action endpoints only when:

* Not idempotent
* Complex workflows (e.g. `/checkout`, `/refund`)

---

## 2. API Versioning Strategy

### Recommended (Production)

```
/api/v1/users
```

But senior insight:

**Avoid versioning unless necessary**

* Prefer **backward compatibility**
* Use **feature flags / additive changes**

---

### Versioning Triggers

Only version when:

* Response contract fundamentally changes
* Business meaning changes (not just structure)

---

## 3. Error Handling & Contracts

### Production-Grade Error Model

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "invalid input",
    "details": [
      {
        "field": "email",
        "reason": "invalid_format"
      }
    ]
  },
  "meta": {
    "request_id": "req-123",
    "trace_id": "trace-xyz"
  }
}
```

---

### Go-Specific Pattern

```go
type AppError struct {
    Code    string
    Message string
    Err     error
}
```

Always:

* Wrap errors (`fmt.Errorf("...: %w", err)`)
* Map internal errors → API errors
* Never leak internal stack traces

---

### Error Design Philosophy

| Principle              | Why                    |
| ---------------------- | ---------------------- |
| Deterministic errors   | Easier client handling |
| Machine-readable codes | Avoid string parsing   |
| Include request_id     | Debugging              |

---

## 4. Pagination at Scale

### Offset Pagination (Basic)

```
GET /users?limit=20&offset=40
```

Problem:

* Slow on large datasets
* Inconsistent under writes

---

### Cursor-Based Pagination (Recommended)

```
GET /users?cursor=eyJpZCI6MTIzfQ==
```

Response:

```json
{
  "data": [...],
  "next_cursor": "abc123"
}
```

---

### Go Insight

Use:

* Indexed fields (`created_at`, `id`)
* Stable ordering

---

## 5. Authentication & Authorization

### JWT Flow (Baseline)

* Access token (short-lived)
* Refresh token (long-lived)

---

### Senior Considerations

#### Token Design

* Keep JWT **small**
* Avoid embedding heavy claims
* Use **opaque tokens** if revocation needed

---

#### Authorization Layer

Do NOT mix in handlers.

Use:

```go
func Authorize(ctx context.Context, userID, resourceID string) error
```

---

#### Multi-Tenant Systems

* Always include `account_id` / `tenant_id`
* Enforce at DB layer (not just API)

---

## 6. Rate Limiting & Resilience

### Strategies

| Strategy       | Use Case      |
| -------------- | ------------- |
| Token bucket   | APIs          |
| Leaky bucket   | Smooth bursts |
| Sliding window | Precision     |

---

### Distributed Rate Limiting

Use:

* Redis
* API Gateway (NGINX / Envoy)

---

### Go Middleware Example

```go
func RateLimit(next http.Handler) http.Handler
```

---

## 7. Observability (Critical for Senior Engineers)

### Must-Have Signals

| Type    | Tool          |
| ------- | ------------- |
| Logs    | Zap / Logrus  |
| Metrics | Prometheus    |
| Traces  | OpenTelemetry |

---

### Golden Signals

* Latency
* Traffic
* Errors
* Saturation

---

### Request Context Propagation

```go
ctx := r.Context()
```

✔ Always pass context:

* DB calls
* external APIs

---

## 8. Concurrency & Performance (Go-Specific)

### Patterns

#### Worker Pool

```go
jobs := make(chan Job)
```

#### Fan-out / Fan-in

```go
go fetchA()
go fetchB()
```

---

### Rules

* Never spawn unbounded goroutines
* Use context cancellation
* Avoid shared memory → prefer channels

---

## 9. Data Layer Design

### Repository Pattern

```go
type UserRepo interface {
    GetByID(ctx context.Context, id string) (*User, error)
}
```

---

### Transactions

```go
tx, err := db.BeginTx(ctx, nil)
```

✔ Use for:

* Multi-step writes
* Financial systems

---

### Idempotency (CRITICAL)

Use:

```
Idempotency-Key: xyz-123
```

Store:

* request + response

---

## 10. Caching Strategy

### Layers

| Layer | Tool        |
| ----- | ----------- |
| CDN   | CloudFront  |
| API   | Redis       |
| DB    | Query cache |

---

### Patterns

* Cache-aside
* Write-through

---

### Cache Key Design

```
user:{id}
orders:user:{id}
```

---

## 11. System Design Thinking

### Always Consider

* What happens under **10x traffic?**
* What breaks first?
* Where is the bottleneck?

---

### Failure Scenarios

* DB down
* External API timeout
* Partial failures

---

### Resilience Patterns

* Retries (with backoff)
* Circuit breaker
* Timeouts

---

## 12. HTTP Semantics (Quick Reference)

| Method | Use                     |
| ------ | ----------------------- |
| GET    | Read                    |
| POST   | Create / non-idempotent |
| PUT    | Replace                 |
| PATCH  | Partial update          |
| DELETE | Remove                  |
