# Sequential Dependency

## Definition

A sequential dependency exists when the value or type of an event depends on the previous event in the sequence.  

Formally, for events `E[i]` and `E[i-1]`, the attribute `B` of `E[i]` is determined by the attribute(s) of `E[i-1]`.

## Rationale

Sequential dependencies model **stateful or causal relationships** between events.  
This is common in event streams where the current action depends on the previous action(s) or system state.  

Unlike functional dependencies, whose correlations are local to a specific event type, sequential dependencies require **shared context**. Knowing previous events of different types is necessary to form the next event.  

## Properties

- the dependency is **order-sensitive**, changing the sequence of events can violate the dependency  
- order of events of different types is deterministic
- the dependency requires **shared context** for events of different types

## Validation query

```sql
CREATE TABLE sequential_dependency
(
    timestamp DateTime,
    session_id UUID,
    type String
)
ENGINE = MergeTree()
ORDER BY (session_id, timestamp);

SELECT * FROM (
    SELECT
        *,
        leadInFrame(toNullable(start_time)) OVER global_w AS next_session_start,
        isNull(next_session_start) AS is_last,
        (IF(is_last, 0, next_session_start < end_time)) AS is_overlapping
    FROM (
        SELECT 
            session_id,
            groupArray(type) OVER inner_w AS path,
            count() OVER inner_w AS count,
            min(timestamp) OVER inner_w AS start_time,
            max(timestamp) OVER inner_w AS end_time
        FROM sequential_dependency
        WINDOW inner_w AS (
            PARTITION BY session_id 
            ORDER BY timestamp ASC 
            ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
        )
        LIMIT 1 BY session_id
    )
    WINDOW global_w AS (ORDER BY start_time ASC ROWS BETWEEN CURRENT ROW AND 1 FOLLOWING)
)
WHERE path != ['start', 'action', 'end'] OR count != 3 OR is_overlapping = 1
ORDER BY start_time ASC
```
