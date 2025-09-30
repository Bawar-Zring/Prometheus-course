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
   rate(http_requests_total[5m])
   ```
   - Use case: Good for stable trend analysis of counters

3. **`irate()`**: Calculates the per-second instantaneous rate of increase using the last two data points
   ```
   irate(http_requests_total[5m])
   ```
   - Use case: Better for highly volatile, fast-moving counters

4. **`increase()`**: Calculates the total increase in a counter over a time range
   ```
   increase(http_requests_total[5m])
   ```
   - Use case: When you need the absolute increase rather than a per-second rate, useful for alerting on total counts

### Delta Functions
4. **`delta()`**: Calculates the difference between first and last value in a time range
   ```
   delta(cpu_usage[5m])
   ```
   - Use case: For gauges to see the overall change in value

6. **`idelta()`**: Calculates the difference between the last two samples
   ```
   idelta(cpu_usage[5m])
   ```
   - Use case: For identifying the most recent change in a gauge

### Over Time Functions
6. **`avg_over_time()`**: Calculates the average value over a time range
   ```
   avg_over_time(cpu_usage[5m])
   ```

7. **`min_over_time()`**: Finds the minimum value over a time range
   ```
   min_over_time(cpu_usage[5m])
   ```

9. **`max_over_time()`**: Finds the maximum value over a time range
   ```
   max_over_time(cpu_usage[5m])
   ```

10. **`sum_over_time()`**: Calculates the sum of all values over a time range
   ```
   sum_over_time(cpu_usage[5m])
   ```

11. **`count_over_time()`**: Counts the number of samples over a time range
    ```
    count_over_time(cpu_usage[5m])
    ```



### Aggregation Functions
12. **`topk()`**: Returns the top k elements with the highest values
    ```
    topk(5, cpu_usage)
    ```
    - Use case: Finding the most resource-intensive containers/services

14. **`bottomk()`**: Returns the bottom k elements with the lowest values
    ```
    bottomk(5, cpu_usage)
    ```
    - Use case: Finding the least resource-intensive containers/services



# Container Monitoring Queries

Common PromQL queries for monitoring containers in production environments:

## Resource Usage

### CPU usage rate
```promql
rate(container_cpu_usage_seconds_total[5m])
```
### CPU throttling
```
rate(container_cpu_cfs_throttled_periods_total[5m])
```
### CPU usage percentage
```
100 * (rate(container_cpu_usage_seconds_total[5m]) / sum(rate(container_cpu_usage_seconds_total[5m])) by (instance))
```

## Memory
### Memory usage in bytes
```promql
container_memory_usage_bytes
```

### Memory usage percentage
```
100 * (container_memory_usage_bytes / container_spec_memory_limit_bytes)
```

## Network I/O Operations

### Network receive throughput
```promql
rate(container_network_receive_bytes_total[5m])
```

### Network transmit throughput
```
rate(container_network_transmit_bytes_total[5m])
```

## Disk I/O
### Disk read throughput
```promql
rate(container_fs_reads_bytes_total[5m])
```

### Disk write throughput
```
rate(container_fs_writes_bytes_total[5m])
```

## Reliability Metrics

### Container restart count
```promql
increase(container_restart_count[5m])
```
### Out of Memory (OOM) kills
```
increase(container_oom_kills_total[5m])
```

---

*Note: These examples assume you have the standard container metrics exporters like cAdvisor running in your environment.*



