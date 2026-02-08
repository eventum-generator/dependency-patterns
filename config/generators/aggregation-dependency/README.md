# Aggregation Dependency

## Definition

An aggregation dependency exists when an attribute of an event is determined by an aggregate function over a bounded set of surrounding events rather than by any single event, current context or state.

Formally, for an event `E[i]`, an attribute `B[i]` is defined as the result of applying an aggregation function `f` to a window of events that includes finite number `k` of preceding events `E[i], E[i-1] ... E[i-k]`.

## Rationale

Many systems exhibit properties that are not observable at the level of single events but become meaningful only when events are considered together within a bounded context. Such dependencies allow modeling intensity, concentration, stability, or structural patterns that arise through accumulation over time or count.

By formalizing how aggregates influence event attributes, aggregation dependencies provide a foundation for reasoning about higher-level behavior in event streams.

## Properties

- the dependency is defined over a bounded set of events rather than over individual events
- the dependent attribute reflects collective characteristics of the window, not properties of any single event
- the dependency is invariant to individual event ordering within the aggregation boundary

## Validation query

```sql
CREATE TABLE aggregation_dependency
(
    timestamp DateTime64(3),
    is_peak UInt8,
    eps Float64
)
ENGINE = MergeTree
ORDER BY timestamp;

SELECT
    count() AS total_events,
    countIf(is_peak) AS peak_events,

    corr(eps, is_peak) AS eps_peak_correlation,

    avgIf(eps, is_peak) AS avg_eps_peak,
    avgIf(eps, NOT is_peak) AS avg_eps_non_peak,

    quantilesExact(0.25, 0.5, 0.75)(eps) FILTER (WHERE is_peak = 1) AS peak_eps_quantiles,
    quantilesExact(0.25, 0.5, 0.75)(eps) AS global_eps_quantiles,

    entropy(is_peak) AS peak_entropy,
FROM aggregation_dependency;
```
