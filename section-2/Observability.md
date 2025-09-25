# Observability in Modern Software Architecture

## Evolution of Application Architecture

### Monolithic Architecture [Waterfall Methodology]
- **Structure**: All services combined in one application
  - UI, business logic, and data access in a single codebase
  - All services use one shared database
- **Challenges**:
  - Entire application needs to be deployed for any change
  - Incremental improvements are time-consuming
  - Scaling becomes difficult as the application grows
  - Single point of failure

### Microservices Architecture [CI/CD]
- **Structure**:
  - Individual services with single responsibility
  - Each service has its own database
  - UI and backend services are separate
- **Benefits**:
  - Easy deployment of small changes
  - Services can scale independently
  - Technology diversity (each service can use appropriate tech stack)
  - Better fault isolation
- **Challenges**:
  - Many services to monitor
  - Intra-service communication may fail
  - Increased attack surface for each service
  - Complex distributed system management

## Monitoring and Observability

### What is Monitoring?
Monitoring is the systematic process of collecting, analyzing, and visualizing data to maintain oversight of systems.

**The monitoring answers three key questions**:
1. Is the service available? (HTTP response 200)
2. Is it functioning as expected? (Error rates)
3. Is the service performing well? (Response times)

### Telemetry Data
Telemetry data refers to the automatic collection, transmission, and remote reception of data from distant sources for monitoring, analysis, and control.

**Important characteristics**:
- Helps identify where problems might be, though not diagnose the root cause itself
- Forms the foundation of observability
- Enables proactive issue detection

### Key Performance Metrics
- **Mean Time to Detection (MTTD)**: Average time between when an incident occurs and when it's detected
- **Mean Time to Resolution (MTTR)**: Average time between incident detection and resolution

## Monitoring Layers and Methods

### 1. UI Layer: Core Web Vitals
These metrics directly impact SEO ranking (poor performance can lower search engine ranking):

- **Largest Contentful Paint (LCP)**: Measures loading performance - how long it takes to render the largest content element visible in the viewport
- **First Input Delay (FID)**: Measures interactivity - the time from when a user first interacts with your page to when the browser can respond to that interaction
- **Cumulative Layout Shift (CLS)**: Measures visual stability - quantifies how much unexpected layout shift occurs during the loading of the page

### 2. Service Layer: RED Method
- **Rate**: Number of requests per second
- **Errors**: Rate of failed requests (HTTP 500s, etc.)
- **Duration**: Response time/latency statistics (average, percentiles)

### 3. Infrastructure Layer: USE Method
- **Utilization**: Percentage of time the resource is busy (CPU, Memory, Disk)
- **Saturation**: Amount of work a resource has to do, often measured by queue length
- **Errors**: Count of error events (disk write errors, network errors, etc.)

## Types of Telemetry Data (MELT)

- **Metrics**: Aggregated numeric values representing system behavior over time
  - Examples: CPU utilization, request count, error rate
  - Good for trends, alerts, and dashboards

- **Events**: Discrete occurrences that happen at a point in time
  - Examples: Deployments, configuration changes, user actions
  - Often processed through systems like Kafka

- **Logs**: Detailed text records of events in the system
  - Examples: Application logs, access logs, error logs
  - Provide context for troubleshooting

- **Traces**: Records of requests as they flow through distributed services
  - Shows interaction between microservices
  - Helps identify bottlenecks and dependencies

## Methods of Collecting Metrics

### Push vs. Pull Model

- **Push Model**:
  - Services actively send metrics to a collector
  - Good for short-lived processes
  - Examples: StatsD, Graphite, Datadog agents

- **Pull Model**:
  - Monitoring system scrapes metrics from service endpoints
  - Allows central control of scrape frequency
  - Example: Prometheus pulls metrics from exporters (Node Exporter, Windows Exporter, etc.)
  - Services expose a `/metrics` endpoint that Prometheus scrapes

## Observability Platforms

- **Prometheus**: Open-source monitoring and alerting toolkit
- **Grafana**: Visualization platform commonly paired with Prometheus
- **ELK Stack** (Elasticsearch, Logstash, Kibana): Log processing and analysis

## Conclusion

Observability is crucial in modern microservice architectures, allowing teams to understand complex distributed systems and quickly address issues. The combination of metrics, events, logs, and traces provides a comprehensive view of application health and performance.