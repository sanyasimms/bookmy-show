# Concurrency Strategy

## Problem

5 lakh users may attempt to book the same seats at the same time.

System requirements:

* Zero double-bookings
* API response under 500ms
* AWS budget under $2000/month

---

# Option A — PostgreSQL Row Locking

Approach:

```sql
BEGIN;

SELECT id
FROM seats
WHERE id=ANY($1)
AND status='available'
FOR UPDATE;

UPDATE seats
SET status='held'
WHERE id=ANY($1);

COMMIT;
```

### Advantages

* Strong ACID guarantees
* Simpler implementation
* No extra infrastructure

### Limitations

Database pool becomes bottleneck.

Connection calculation:

Connections held =
(0.8 × RPS × 0.02)
+
(0.2 × RPS × 0.8)

500 =
0.176 × RPS

RPS ≈ 2840

Meaning:
Pool exhaustion occurs near 2840 requests/sec.

Expected traffic ≈ 16K+ RPS.

Deadlock mitigation:
Always lock seats in sorted order.

---

# Option B — Redis SETNX Lock

Lock Key:

seat_lock:{event_id}:{seat_id}

Acquire:

```javascript
redis.set(
 lockKey,
 value,
 'NX',
 'EX',
 30
)
```

Release:

Lua script verifies ownership before delete.

### Advantages

* Very high throughput
* Reduces DB pressure
* Fast lock acquisition

### Risks

* Redis adds infrastructure cost
* Incorrect TTL can create stale locks

TTL decisions:

* Redis acquire lock → 30 sec
* Seat hold → 10 min

---

# Final Choice — Hybrid

Redis handles seat contention.

PostgreSQL confirms final booking.

Reason:

* Supports 5L concurrency
* Fits AWS budget
* Preserves correctness

Switch condition:

If traffic drops below ~3000 RPS,
move fully to PostgreSQL.
