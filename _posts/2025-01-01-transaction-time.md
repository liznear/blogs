---
title: "Ordering by Transaction Time"
date: 2025-01-01
layout: post
---

Using an RDBMS table as a queue is a common practice. At work, we have a queue table in a PostgreSQL database with a `created` column indicating when each row was inserted. Multiple “publishers” add items to this queue, and to ensure that the created column is consistently sortable, we use `now()` rather than relying on the local time of the publishers.

A single consumer processes this queue using pseudocode like the following:

```go
offset := some_time
for {
  rows := db.QueryContext(ctx, "SELECT * FROM queue WHERE created > $1", offset)
  // consume rows
  offset = maxCreated(rows)
}
```

This looks straightforward, but there’s a subtle bug.

## The Bug

At time T1, the loop might see a row with `created = 2025-01-01 00:00:00` and update offset to this value. This assumes that all rows with `created <= 2025-01-01 00:00:00` have already been added to the table and consumed.

However, this assumption isn’t always valid. A transaction that started at `2024-12-31 23:59:59 ` might not have been committed yet. When that transaction is eventually committed, the new row (`created = 2024-12-31 23:59:59`) will be missed because the offset has already moved past it.

## Add a Delay Before Reading

One practical solution is to add a slight delay to the most recent data being read. For example:

```go
offset := some_time
for {
  rows := db.QueryContext(ctx, `
SELECT *
FROM queue
WHERE created > $1
  AND created < (
    -- 5 seconds is the delay
    SELECT max(created) - '5 seconds'::interval
    FROM queue
  )
`, offset)
  // consume rows
  offset = maxCreated(rows)
}
```

By introducing a 5-second buffer, we ensure that all transactions committed within that window are included in the results. As long as transactions take less than 5 seconds to commit, this issue won’t occur.

The trade-off here is the slight delay between publishing and consuming events.

## Other Alternatives

### Use Transaction Commit Time

Another approach is to use the transaction commit time (e.g. `pg_xact_commit_timestamp(xmin)` in PostgreSQL). Since this timestamp is only available once the transaction is committed, it requires an additional process to update the created column with the commit time.

This extra process must be highly available and robust. Otherwise, rows in the queue table won't have this `created` column set and thus won't be processed. In my opinion, the added operational complexity outweighs the benefits.

### Use an Auto-Increment Sequence

You might think that ordering by an auto-incrementing sequence solves the issue. However, it suffers from the same fundamental problem. When a transaction begins, the sequence value is assigned immediately. Therefore, a later-committed transaction can have a smaller sequence ID.

## Watermark in Streaming Processing

Our delayed-offset approach resembles the watermark mechanism commonly used in streaming processing systems.

In streaming systems, there are typically two types of time:

- **Event Time**: When the event actually happened.
- **Processing Time**: When the system received and processed the event.

Just like our problem, events that happened earlier might arrive later. Missing such “late events” can cause inconsistencies or incorrect results.

To address this, streaming systems introduce a watermark — a timestamp indicating that all events with an earlier event time are assumed to have already arrived.

A simple watermarking strategy is to set it as `max(event_time) - some_delay`.

This is effectively what our solution does. By introducing a small, controlled delay, we ensure correctness while balancing latency and consistency.
