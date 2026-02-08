# State dependency

## Definition

A state dependency exists when the type or attributes of an event is determined by the state in which the event occurs.

## Rationale

State dependencies model systems where events are emitted according to the current state of the system rather than solely as a function of input or time.

In such systems, the same triggering conditions may produce different events depending on the active state.
State transitions are governed by explicit conditions, and once a state is entered, it constrains the set of valid event types or attribute values.

## Properties

- event attributes are constrained by the current state
- the state of the current event is a direct consequence of the state and attribute values of the preceding event
- transitions between states can be deterministic or probabilistic

## Validation query

```sql
CREATE TABLE state_dependency
(
    timestamp DateTime,
    progress UInt8,
    stage String
)
ENGINE = MergeTree()
ORDER BY (timestamp);

SELECT 
    timestamp,
    stage,
    progress,
    prev_stage,
    prev_progress,
    CASE 
        WHEN stage = 'done' AND prev_progress != 99 
            THEN 'Invalid success transition: expected progress 99'

        WHEN prev_stage = 'failure' AND progress != 0 
            THEN 'Post-failure reset failed: progress must be 0'

        WHEN stage = 'in progress' AND prev_stage = 'in progress' AND progress != prev_progress + 1
            THEN 'Counter anomaly: expected monotonic increment'

        WHEN prev_stage = 'done' AND stage != 'in progress'
            THEN 'State flow broken: expected in progress after done'
        
        ELSE 'OK'
    END AS validation_result
FROM (
    SELECT 
        timestamp,
        stage,
        progress,
        lagInFrame(stage) OVER w AS prev_stage,
        lagInFrame(progress) OVER w AS prev_progress
    FROM state_dependency
    WINDOW w AS (ORDER BY timestamp ASC)
)
WHERE validation_result != 'OK'
ORDER BY timestamp ASC
```
