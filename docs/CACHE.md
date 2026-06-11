# Cache Design

## Cache Strategy

Pattern used:
**Cache Aside**

Flow:

DB → Delete Cache → Next Read → Repopulate Cache

---

# Event Details

Redis Key:

event:{event_id}

Stored:

* event name
* venue
* start time
* status

TTL:
3600 seconds

Invalidation:

Trigger:
events table UPDATE

Reason:

Event metadata changes rarely.

---

# Seat Availability Count

Redis Key:

availability:{event_id}:{category}

Stored:

Available seat count.

TTL:
30 seconds

Invalidation:

Whenever any seat status changes.

Example:

available → held

held → booked

held → available

Reason:

High read frequency.

Small staleness acceptable.

---

# Seat Map

Redis Key:

seatmap:{event_id}

Stored:

Static seat structure.

TTL:
86400 seconds

Invalidation:

Event cancellation.

Reason:

Layout rarely changes.

---

# Explicitly NOT Cached

Individual seat availability.

Reason:

Can create stale reads.

Two users could see same seat.

Booking correctness must come from:

* Redis lock state
* PostgreSQL source of truth

---

# Invalidation Strategy

Event-driven invalidation.

Pseudo Flow:

1. Update DB
2. Delete cache key
3. Next request repopulates

Pseudo Code:

```javascript
updateSeat()

redis.del(
`availability:${eventId}:${category}`
)
```

Reason:

Keeps counts fresh.

Avoids long stale windows.
