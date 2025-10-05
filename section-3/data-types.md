# Prometheus Data Types and Query Language Reference

This document provides an overview of Prometheus data types, operators, functions, and common query patterns for monitoring applications.

## Data Types

Prometheus uses several fundamental data types for its time series database:

### Basic Data Types

- **Scalar**: A simple numeric floating-point value
- **String**: A string value (used in limited contexts)
- **Instant Vector**: A set of time series, each containing a single sample for a specific timestamp
    ```
    container_cpu_usage_seconds_total
    ```
- **Range Vector**: A set of time series containing all samples over a specified time range
    ```
    container_cpu_usage_seconds_total[5m]
    ```

## Metric Types

Prometheus metrics can be of four types:
- **Counter**: A cumulative metric that increases over time (e.g., `http_requests_total`)
- **Gauge**: A value that can go up or down (e.g., `memory_usage_bytes`)
- **Histogram**: Samples observations into configurable buckets and counts them (e.g., request latency)
- **Summary**: Similar to histogram but provides quantiles over a sliding time window

## Operators

### Arithmetic Operators
- Addition: `+`
- Subtraction: `-`
- Multiplication: `*`
- Division: `/`
- Modulo: `%`
- Power: `^`

### Comparison Operators
- Equal: `==`
- Not equal: `!=`
- Greater than: `>`
- Less than: `<`
- Greater or equal: `>=`
- Less or equal: `<=`
```
rate(container_cpu_usage_seconds_total{name=~"app|db|nginx"}[5m])
```
### Logical/Set Operators
- `and`: Intersection of two instant vectors
- `or`: Union of two instant vectors
- `unless`: Set difference (elements in first vector that aren't in second)

## Aggregation Operators

- `sum`: Sum over dimensions
- `avg`: Average over dimensions
- `min`: Minimum over dimensions
- `max`: Maximum over dimensions
- `count`: Count number of elements
- `count_values`: Count values for each unique sample value
- `stddev`: Standard deviation over dimensions
- `stdvar`: Standard variance over dimensions
- `topk`: Top k elements by sample value
- `bottomk`: Bottom k elements by sample value

### Modifier Keywords
- `by`: Groups results by specified labels
- `without`: Groups results by all labels except those specified
- `on`: Matching labels for vector operations
- `ignoring`: Ignores specified labels for vector operations

## Time Offset Modifier

The `offset` modifier shifts the time range of a query by a specified duration:
    ```
    container_cpu_usage_seconds_total offset 5m
    ```
- Usage: This retrieves data as it was 5 minutes ago

## Common Prometheus Functions

### Rate Functions
1. **`rate()`**: Calculates the per-second average rate of increase of a counter over a time range
   ```
   rate(container_cpu_usage_seconds_total[5m])
   ```
   - Use case: Good for stable trend analysis of counters

3. **`irate()`**: Calculates the per-second instantaneous rate of increase using the last two data points
   ```
   irate(container_cpu_usage_seconds_total[5m])
   ```
   - Use case: Better for highly volatile, fast-moving counters

4. **`increase()`**: Calculates the total increase in a counter over a time range
   ```
   increase(container_cpu_usage_seconds_total[5m])
   ```
   - Use case: When you need the absolute increase rather than a per-second rate, useful for alerting on total counts

### Delta Functions
5. **`delta()`**: Calculates the difference between first and last value in a time range
   ```
   delta(container_cpu_usage_seconds_total[5m])
   ```
   - Use case: For gauges to see the overall change in value

6. **`idelta()`**: Calculates the difference between the last two samples
   ```
   idelta(container_cpu_usage_seconds_total[5m])
   ```
   - Use case: For identifying the most recent change in a gauge

### Over Time Functions
7. **`avg_over_time()`**: Calculates the average value over a time range
   ```
   avg_over_time(container_cpu_usage_seconds_total[5m])
   ```

8. **`min_over_time()`**: Finds the minimum value over a time range
   ```
   min_over_time(container_cpu_usage_seconds_total[5m])
   ```

9. **`max_over_time()`**: Finds the maximum value over a time range
   ```
   max_over_time(container_cpu_usage_seconds_total[5m])
   ```

10. **`sum_over_time()`**: Calculates the sum of all values over a time range
   ```
   sum_over_time(container_cpu_usage_seconds_total[5m])
   ```

11. **`count_over_time()`**: Counts the number of samples over a time range
    ```
    count_over_time(container_cpu_usage_seconds_total[5m])
    ```

# Prometheus: Aggregation vs Over-Time Functions

| Feature                | Aggregation Functions                   | Over-Time Functions                           |
|------------------------|------------------------------------------|-----------------------------------------------|
| **Purpose**            | Combine **multiple series** at one moment | Summarize **one series** over a past time window |
| **Input Type**         | **Instant vector** (current values)      | **Range vector** (historical samples)         |
| **Output Type**        | **Instant vector** (one value per group) | **Instant vector** (one summarized value per series) |
| **Common Functions**   | `sum`, `avg`, `max`, `min`, `count`      | `avg_over_time`, `min_over_time`, `max_over_time`, `sum_over_time` |
| **Works Best With**    | Both **gauges** and counters (after `rate()`) | Gauges directly, or counters after `rate()` / `increase()` |
| **Typical Question**   | “What’s the total/average **now** across all pods?” | “What was the average/max **over the last 5m** for each pod?” |
| **Example**            | `sum(rate(http_requests_total[5m])) by (pod)` | `avg_over_time(container_cpu_usage_seconds_total[5m])` |

👉 **Shortcut:**  
- **Aggregation → across series, now**  
- **Over-Time → across past samples, in one series**
