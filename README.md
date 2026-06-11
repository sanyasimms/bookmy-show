# ShowTime - BookMyShow Architecture Design

## Constraint Analysis

### Constraint 1: 5 Lakh Concurrent Users

Expected traffic spike occurs at 12:00:00 noon.

Assumption:

* Average 2 API requests per user during first minute

Estimated RPS:

RPS = (500000 × 2) ÷ 60
≈ 16,667 requests/sec

Expected bottlenecks:

* Database connection pool
* Seat locking contention
* Payment processing latency

---

### Constraint 2: Zero Double Booking

Double booking means multiple confirmed bookings for the same seat.

Prevention strategy:

* Redis distributed locking
* PostgreSQL transaction validation
* Optimistic locking using version field

---

### Constraint 3: AWS Budget ($2000/month)

Infrastructure priorities:

* API servers
* PostgreSQL database
* Redis cache
* SQS queue

If budget dropped to $500:

* Reduce replicas
* Lower Redis capacity
* Increase cache usage

---

## Repository Structure

docs/

* SCHEMA.md
* CONCURRENCY.md
* CACHE.md
* QUEUE.md
* ARCHITECTURE.md
* DESIGN-DECISIONS.md
* DESIGN-UPDATES.md
