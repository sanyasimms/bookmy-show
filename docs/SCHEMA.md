# Database Schema

## venues

```sql
CREATE TABLE venues (
 id SERIAL PRIMARY KEY,
 name VARCHAR(255) NOT NULL,
 city VARCHAR(100) NOT NULL,
 capacity INT NOT NULL CHECK(capacity>0)
);

CREATE INDEX idx_venue_city
ON venues(city);
```

---

## events

```sql
CREATE TABLE events (
 id SERIAL PRIMARY KEY,
 venue_id INT NOT NULL
 REFERENCES venues(id)
 ON DELETE CASCADE,

 name VARCHAR(255) NOT NULL,

 start_time TIMESTAMP NOT NULL,

 status VARCHAR(20)
 CHECK (
 status IN (
 'upcoming',
 'on_sale',
 'sold_out',
 'cancelled'
 )
 ),

 total_seats INT
 CHECK(total_seats>0)
);

CREATE INDEX idx_events_status
ON events(status);
```

---

## users

```sql
CREATE TABLE users (
 id UUID PRIMARY KEY,

 email VARCHAR(255)
 UNIQUE NOT NULL,

 phone VARCHAR(20)
 UNIQUE NOT NULL,

 name VARCHAR(255)
 NOT NULL,

 created_at TIMESTAMP
 DEFAULT CURRENT_TIMESTAMP
);
```

---

## seats

```sql
CREATE TABLE seats (

 id SERIAL PRIMARY KEY,

 event_id INT
 REFERENCES events(id)
 ON DELETE CASCADE,

 section VARCHAR(20),

 row_name VARCHAR(20),

 seat_number INT,

 price DECIMAL(10,2)
 CHECK(price>0),

 category VARCHAR(20)
 CHECK(
 category IN(
 'VIP',
 'General',
 'Premium'
 )
 ),

 status VARCHAR(20)
 DEFAULT 'available'
 CHECK(
 status IN(
 'available',
 'held',
 'booked'
 )
 ),

 held_until TIMESTAMP,

 held_by UUID
 REFERENCES users(id)
 ON DELETE SET NULL,

 version INT DEFAULT 0
);

CREATE INDEX idx_seats_event_status
ON seats(event_id,status);
```

---

## bookings

```sql
CREATE TABLE bookings (

 id UUID PRIMARY KEY,

 user_id UUID
 REFERENCES users(id)
 ON DELETE CASCADE,

 event_id INT
 REFERENCES events(id),

 status VARCHAR(20)
 CHECK(
 status IN(
 'pending',
 'confirmed',
 'failed',
 'refunded'
 )
 ),

 total_amount DECIMAL(10,2)
 CHECK(total_amount>0),

 payment_reference TEXT,

 created_at TIMESTAMP
 DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_bookings_user
ON bookings(
user_id,
created_at DESC
);

CREATE INDEX idx_bookings_pending
ON bookings(status)
WHERE status IN(
'pending',
'failed'
);
```

---

## booking_seats

```sql
CREATE TABLE booking_seats (

 booking_id UUID
 REFERENCES bookings(id)
 ON DELETE CASCADE,

 seat_id INT
 REFERENCES seats(id)
 ON DELETE RESTRICT,

 PRIMARY KEY(
 booking_id,
 seat_id
 )
);
```

---

# Design Decisions

## Why UUID?

Prevents predictable booking enumeration and supports distributed generation.

---

## Why version column?

Supports optimistic locking to detect concurrent seat updates.

---

## Why held_until?

Automatically releases abandoned seats.

---

## Why partial index?

Speeds unresolved booking lookup while keeping index size small.
