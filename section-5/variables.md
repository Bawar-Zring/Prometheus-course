# Variables in Prometheus

Variables in Prometheus are used to store values that can be reused throughout your queries and configurations. They make your queries more dynamic, flexible, and easier to manage, especially in dashboarding tools like Grafana.

## Types of Variables

### 1. Query Variables
These variables are defined by a Prometheus query and dynamically populate options in dashboards or alerts. They're powerful for allowing users to select from available metrics or labels.

### 2. Constant Variables
Fixed values that don't change during dashboard usage. They're useful for setting thresholds or specific values in your queries.

### 3. Interval Variables
Define time intervals that can be used in queries to specify the range of data to be retrieved. These are particularly useful for rate() and increase() functions.

### 4. Custom Variables
User-defined variables that can be set to any value and used throughout your queries and configurations. Useful for defining environments, regions, or other custom parameters.

### 5. Textbox Variables
Allow users to input custom values through a text box in the dashboard. Useful when you need specific input that can't be pre-defined.

### 6. Data Source Variables
Used to select different data sources in your queries. Particularly useful in multi-datasource environments.
  
## Best Practices for Variables

1. **Use descriptive names** - Choose clear, descriptive names for your variables to make queries more readable
2. **Include meaningful defaults** - Always set useful default values for your variables
3. **Use regex filtering** - Take advantage of regex in your variable definitions to filter out unwanted values
4. **Group related variables** - Organize variables that work together into logical groups
5. **Consider dependency chains** - Set up cascading variables where one variable's value depends on another

## Variable Syntax

- In Prometheus and PromQL, variables are referenced with a `$` prefix: `$variable_name`
- In some contexts (like interval variables), you may need to use `${variable_name}` syntax
- For multi-value variables, use regex matchers: `{label=~"$variable"}`

## Common Variable Use Cases

### Dashboard Template Variables
Create templated dashboards that can be reused across different instances, environments, or services:

```
sum by(service) (rate(http_requests_total{environment="$env", service=~"$service"}[$interval]))
```

### Dynamic Time Ranges
Allow users to select different time ranges for analysis:

```
increase(http_requests_total[$time_range])
```

These variables significantly enhance the flexibility and usability of Prometheus queries, enabling the creation of dynamic and interactive dashboards and alerts that can adapt to different monitoring scenarios.