# Promtail — example configuration with per-block explanations

This document shows a complete Promtail configuration and explains each block briefly after the full config. Use the config as a starting point and update the placeholders (Loki URL, paths, selectors) for your environment.

## Full configuration (ready-to-use example)

### server
```yml
  http_listen_port: 9080
  grpc_listen_port: 0
```

---

### clients
```yml
  - url: http://<LOKI-SERVER>:3100/loki/api/v1/push

    # Maximum amount of time to wait before sending a batch, even if that
    # batch isn't full.
    [batchwait: <duration> | default = 1s]

    # Maximum batch size (in bytes) of logs to accumulate before sending
    # the batch to Loki.
    [batchsize: <int> | default = 1048576]

    # Configures how to retry requests to Loki when a request
    # fails.
    # Default backoff schedule:
    # 0.5s, 1s, 2s, 4s, 8s, 16s, 32s, 64s, 128s, 256s(4.267m)
    # For a total time of 511.5s(8.5m) before logs are lost
    backoff_config:
      # Initial backoff time between retries
      [min_period: <duration> | default = 500ms]
    
      # Maximum backoff time between retries
      [max_period: <duration> | default = 5m]
    
      # Maximum number of retries to do
      [max_retries: <int> | default = 10]

    # Maximum time to wait for a server to respond to a request
    [timeout: <duration> | default = 10s]
```

---

### positions
  The positions block configures where Promtail will save a file indicating how far it has read into a file. It is needed for when Promtail is restarted to allow it to continue from where it left off.
```yml
# How often to update the positions file
[sync_period: <duration> | default = 10s]

# Whether to ignore & later overwrite positions files that are corrupted
[ignore_invalid_yaml: <boolean> | default = false]

```
---


### scrape_configs
The scrape_configs block configures how Promtail can scrape logs from a series of targets using a specified discovery method. Promtail uses the same Prometheus scrape_configs. This means if you already own a Prometheus instance, the config will be very similar:

```yml
# Name to identify this scrape config in the Promtail UI.
job_name: <string>

# Describes how to transform logs from targets.
[pipeline_stages: <pipeline_stages>]

# Defines decompression behavior for the given scrape target.
decompression:
  # Whether decompression should be tried or not.
  [enabled: <boolean> | default = false]

# Describes how to relabel targets to determine if they should
# be processed.
relabel_configs:
  - [<relabel_config>]

# Static targets to scrape.
static_configs:
  - [<static_config>]

# Files containing targets to scrape.
file_sd_configs:
  - [<file_sd_configs>]

# Describes how to use the Docker daemon API to discover containers running on
# the same host as Promtail.
docker_sd_configs:
  [ - <docker_sd_config> ... ]

```


### pipeline_stages
Pipeline stages are used to transform log entries and their labels. The pipeline is executed after the discovery process finishes. The pipeline_stages object consists of a list of stages which correspond to the items listed below.
for more informatin look at this link:
https://gitee.com/mirrors/Grafana-Loki/blob/791262dc258e74a5457dedc82879d383cce2e66e/docs/clients/promtail/pipelines.md#pipelines

In most cases, you extract data from logs with regex or json stages. The extracted data is transformed into a temporary map object. The data can then be used by Promtail e.g. as values for labels or as an output. Additionally any other stage aside from docker and cri can access the extracted data.
```yml
    <docker> 
    <cri> 
    <regex> 
    <json> 
    <template> 
    <match> 
    <timestamp> 
    <output> 
    <labels> 
    <metrics> 
    <tenant> 
    <replace>

# The Docker stage parses the contents of logs from Docker containers, and is defined by name with an empty object:
docker: {}

- json:
    expressions:
      output: log
      stream: stream
      timestamp: time
- labels:
    stream:
- timestamp:
    source: timestamp
    format: RFC3339Nano
- output:
    source: output

# regex:
# The Regex stage takes a regular expression and extracts captured named groups to be used in further stages.

regex:
  # The RE2 regular expression. Each capture group must be named.
  expression: <string>

  # Name from extracted data to parse. If empty, uses the log message.
  [source: <string>]

# json
# The JSON stage parses a log line as JSON and takes JMESPath expressions to extract data from the JSON to be used in further stages.

json:
  # Set of key/value pairs of JMESPath expressions. The key will be
  # the key in the extracted data while the expression will be the value,
  # evaluated as a JMESPath from the source data.
  expressions:
    [ <string>: <string> ... ]

  # Name from extracted data to parse. If empty, uses the log message.
  [source: <string>]


# output
# The output stage takes data from the extracted map and sets the contents of the log entry that will be stored by Loki.

output:
  # Name from extracted data to use for the log entry.
  source: <string>


# labels
# The labels stage takes data from the extracted map and sets additional labels on the log entry that will be sent to Loki.

labels:
  # Key is REQUIRED and the name for the label that will be created.
  # Value is optional and will be the name from extracted data whose value
  # will be used for the value of the label. If empty, the value will be
  # inferred to be the same as the key.
  [ <string>: [<string>] ... ]


# replace
# The replace stage is a parsing stage that parses a log line using a regular expression and replaces the log line.

replace:
  # The RE2 regular expression. Each named capture group will be added to extracted.
  # Each capture group and named capture group will be replaced with the value given in
  # `replace`
  expression: <string>

  # Name from extracted data to parse. If empty, uses the log message.
  # The replaced value will be assigned back to soure key
  [source: <string>]

  # Value to which the captured group will be replaced. The captured group or the named
  # captured group will be replaced with this value and the log line will be replaced with
  # new replaced values. An empty value will remove the captured group from the log line.
  [replace: <string>]



```
