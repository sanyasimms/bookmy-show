# Async Order Processing

## Why Async?

Payment APIs take:

200–2000ms

Holding DB connections during payment causes pool exhaustion.

Connection formula:

Connections =
(0.8 × RPS × 0.02)
+
(0.2 × RPS × 0.8)

At high RPS the DB collapses.

Solution:

Queue payment separately.

---

# Queue Message Format

```json
{
"bookingId":"uuid",
"userId":"uuid",
"eventId":101,
"seatIds":[1,2,3],
"totalAmount":4500,
"paymentToken":"token",
"idempotencyKey":"uuid"
}
```

Field Purpose:

bookingId → identify booking

userId → ownership

eventId → context

seatIds → booking details

totalAmount → payment

paymentToken → gateway processing

idempotencyKey → avoid duplicates

---

# Worker Flow

## Success

1. Receive message
2. Call payment gateway
3. Update booking → confirmed
4. Update seats → booked
5. Send notification
6. Delete SQS message

---

## Failure

1. Receive message
2. Retry payment
3. Mark booking failed
4. Release seats
5. Notify user
6. Delete message

---

# Edge Cases

## API crashes after SQS publish

Booking exists.

User may retry.

Idempotency key prevents duplicates.

Worker still completes processing.

---

## Payment timeout

Worker retries.

Maximum retries = 3.

Then move to DLQ.

---

# SQS Configuration

Visibility Timeout:

30 seconds

Reason:

2× expected payment duration.

Max Receive Count:

3

After 3 failures:

Move to Dead Letter Queue.

DLQ monitored for investigation.
