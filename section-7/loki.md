# Loki configuration — annotated

This file documents a sample Loki configuration and explains the important settings. The sample is suitable for local testing and learning. For production use, review the "Production notes" section and the official documentation: https://grafana.com/docs/loki/latest/.

---

## Example Loki configuration (annotated)

This example uses boltdb-shipper with filesystem object store and an in-memory ring — good for single-node or local testing.

```
auth_enabled: false
```
### Global / top-level: auth_enabled
- Purpose: Enables built-in authentication checks in Loki. When `false`, Loki does not enforce auth for requests.
- Typical: `false` for local testing. `true` or use an external proxy (OAuth/OIDC/reverse-proxy) for production.
 
---

```
server:
  http_listen_port: 3100
```
### server
- `http_listen_port`:
  - The HTTP API port used by clients (Promtail, Grafana, curl). Default is `3100`.
  - If you run multiple Loki instances on the same host, use different ports or bind to different addresses.
- Additional common option:
  - `http_listen_address`: bind address (0.0.0.0 or 127.0.0.1).

  ---

```
ingester:
  lifecycler:
    ring:
      kvstore:
        store: inmemory  
      replication_factor: 1
  chunk_idle_period: 5m   
  max_chunk_age: 1h      
  wal:
    dir: /loki/wal       
```

### ingester
The ingester receives write requests and stores log chunks temporarily before they are persisted. Important fields:

- `lifecycler.ring.kvstore.store`:
  - Controls how ingesters coordinate and keep ring membership.
  - Options:
    - `inmemory`: Single-node/testing only — no cross-node membership.
    - `memberlist`: Uses a gossip protocol for node discovery. Good for small clusters and easier than consul.
    - `consul`: Uses Consul KV for membership; useful in environments already using Consul.
  - Example: Use `memberlist` in production clusters where you want automatic gossip-based discovery.

- `replication_factor`:
  - The number of ingesters that will hold each series/chunk in memory and WAL for durability.
  - For HA, set `replication_factor` > 1 and run at least that many ingesters.

- `chunk_idle_period`:
  - How long an open chunk remains idle before it is cut and stored.
  - Lower values flush chunks faster (lower ingestion latency), but cause more index activity and disk I/O.
  - Example: For high-throughput systems, 5m–15m is common. For low-latency requirements, 1m–5m.

- `max_chunk_age`:
  - Maximum duration a chunk may remain open before force rotation.
  - Ensures periodic chunk rotation even when traffic is constant.
  - Typical: 1h to 6h depending on query and storage trade-offs.

- `wal.dir`:
  - Path to write-ahead log (WAL) files that give data durability between ingest and persist stages.
  - Keep this on fast durable disk (SSD) and ensure enough disk space.
 
---

```
schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem   
      schema: v13
      index:
        prefix: index_
        period: 24h
```
### schema_config
- `configs`:
  - A list of schema versions/time ranges. Each entry defines how data for certain times is stored/indexed.
  - Field breakdown:
    - `from`: effective start date for that schema config (YYYY-MM-DD). Loki reads the list and chooses the matching config by time.
    - `store`: `boltdb-shipper` is the recommended scalable indexing mode for many setups.
    - `object_store`: `filesystem`, `s3`, `gcs`, or `azure` — where boltdb-shipper stores objects (index files).
    - `schema`: version of Loki's schema (v9, v10, v11, etc.; v13 in the example).
    - `index.prefix` and `index.period`: control index file naming and rotation. `period: 24h` means index files rotate every 24 hours.
- Why `boltdb-shipper`:
  - It builds small local boltdb index files and uploads them to an object store. This reduces pressure on large centralized indexes and scales better when using object storage.

---
```
storage_config:
  boltdb_shipper:
    active_index_directory: /loki/index
    cache_location: /loki/cache
  filesystem:
    directory: /loki/chunks
```
### storage_config
- `boltdb_shipper.active_index_directory`:
  - Local directory where the current active index files are kept.
  - Must be on fast disk and persisted between restarts.

- `boltdb_shipper.cache_location`:
  - Local cache for downloaded index files to speed up queries.

- `filesystem.directory`:
  - Where chunk files are kept when using `filesystem` object store. For production, use S3/GCS/Azure and configure the `object_store` block accordingly.

---
```
limits_config:
  ingestion_rate_mb: 4                # throttle ingestion (MB/s)
  max_concurrent_tail_requests: 20
  allow_structured_metadata: false
```
### limits_config
Controls throttling and query/ingestion protections.

- `ingestion_rate_mb`:
  - Global ingestion throttle in MB/s. Helps prevent bursts from overwhelming the cluster.
  - Tuning: set slightly above expected sustained ingestion. For example, 4 MB/s in the example for a small environment.

- `max_concurrent_tail_requests`:
  - Limits the number of clients tailing logs (e.g., Grafana live tail) concurrently.

- `allow_structured_metadata`:
  - Controls whether structured metadata (JSON fields) are allowed as metadata. Setting `false` can simplify indexing and reduce cardinality issues.
 
---
```

compactor:
  working_directory: /loki/compactor
  retention_enabled: false            
```
### compactor
Used with `boltdb-shipper` to compact index files and enforce retention policies.

- `working_directory`:
  - Local dir for temporary files for compactor work.

- `retention_enabled`:
  - When `true`, compactor removes old index files and enforces retention slices — requires object store to be configured and compactor properly scheduled.
- Production: run compactor as a separate component (service) with access to the same object store.
---


## How data flows (brief)
1. Promtail (or another client) pushes logs to the Loki HTTP push API on the `server` port (`/loki/api/v1/push`).
2. Receivers/distributor accept the push, shard series and pass requests to ingesters.
3. Ingester stores logs in in-memory chunks and WAL for durability.
4. Periodically, chunks are flushed to object storage (and index files are shipped if using boltdb-shipper) or stored on local filesystem.
5. Compactor (when present) compacts index files and applies retention rules in the object store.
6. Querier reads indexes and chunks (from local cache or object store) to satisfy queries.

---

