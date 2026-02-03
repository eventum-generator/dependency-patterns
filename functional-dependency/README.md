# Functional dependency

## Definition

A functional dependency exists when the value of one attribute uniquely determines the value of another attribute.
Formally: for any two records, if A = x then B = y.

## Rationale

This dependency models deterministic relationships where one value acts as a key and the other as a strictly dependent attribute. Once a mapping between `determinant_value` and `dependent_value` is established, it must remain stable across all generated records.

This is the most basic and strongest form of dependency and serves as a baseline invariant for all more complex cases.

## Properties

- each `determinant_value` is associated with exactly one `dependent_value`
- the same `determinant_value` always produces the same `dependent_value`
- the dependency is global and unconditional

## Validation query

```sql
CREATE TABLE functional_dependency
(
    determinant_value UInt32,
    dependent_value String,
)
ENGINE = MergeTree()
ORDER BY determinant_value;

SELECT
    determinant_value,
    countDistinct(dependent_value) AS distinct_dependent_values
FROM functional_dependency
GROUP BY determinant_value
HAVING distinct_dependent_values > 1
```
