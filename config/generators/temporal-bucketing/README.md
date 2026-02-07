# Functional dependency

## Definition

Temporal bucketing is a deterministic dependency where a categorical attribute is derived directly from the event timestamp by mapping continuous time into discrete intervals (buckets).

Formally, for any record with timestamp t, the value B is defined as B = f(t), where f is a fixed, deterministic time-based function.

## Rationale

This pattern models situations where event attributes are not independent values but are derived from time itself, such as time-of-day, business hours, or operational periods.

Unlike true temporal dependencies, temporal bucketing does not rely on historical context or previous events. Each record is evaluated independently based solely on its timestamp.

## Properties

- the derived attribute `dependent_value` is fully determined by the event timestamp
- the dependency is deterministic and stateless
- event ordering does not affect the result

## Validation query

```sql
CREATE TABLE temporal_bucketing
(
    timestamp DateTime,
    dependent_value String
)
ENGINE = MergeTree()
ORDER BY timestamp;

WITH
    toHour(timestamp) AS hour,
    arraySort(groupUniqArray(hour)) AS covered_hours
SELECT
    dependent_value AS bucket,
    min(hour) AS min_hour,
    max(hour) AS max_hour,
    count() AS events,
    covered_hours,
FROM temporal_bucketing
GROUP BY bucket
ORDER BY min_hour;
```
