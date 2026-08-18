# Delta Lake Transaction Logs (_delta_log/*.json)

## Overview

Transaction logs are JSON files stored in `_delta_log/` directory of each Delta table. They record:
- **commitInfo**: Operation (WRITE, MERGE, RESTORE, etc.) with metrics
- **protocol**: Reader/writer versioning
- **metaData**: Table schema, partitions, creation time
- **add/remove**: File changes (path, size, stats, partition values)

---

## Example 1: Bronze LLM Calls (WRITE - 200K rows)

**Path**: `_lakehouse/bronze/llm_calls_raw/_delta_log/00000000000000000000.json`

```json
{
  "commitInfo": {
    "timestamp": 1787054771844,
    "operation": "WRITE",
    "operationParameters": {
      "mode": "Overwrite"
    },
    "engineInfo": "delta-rs:py-1.6.2",
    "operationMetrics": {
      "execution_time_ms": 190,
      "num_added_files": 1,
      "num_added_rows": 200000,
      "num_partitions": 0,
      "num_removed_files": 0
    },
    "clientVersion": "delta-rs.py-1.6.2"
  }
}

{
  "protocol": {
    "minReaderVersion": 1,
    "minWriterVersion": 2
  }
}

{
  "metaData": {
    "id": "787287fb-cc1d-4f65-bda4-1c504a4d177b",
    "name": null,
    "description": null,
    "format": {
      "provider": "parquet",
      "options": {}
    },
    "schemaString": "{\"type\":\"struct\",\"fields\":[{\"name\":\"request_id\",\"type\":\"string\",\"nullable\":true,\"metadata\":{}},{\"name\":\"ts\",\"type\":\"timestamp\",\"nullable\":true,\"metadata\":{}},{\"name\":\"raw_json\",\"type\":\"string\",\"nullable\":true,\"metadata\":{}}]}",
    "partitionColumns": [],
    "createdTime": 1787054756858,
    "configuration": {}
  }
}

{
  "add": {
    "path": "part-00000-31c31d48-d2f5-4c90-bd52-e854a9df4c14-c000.snappy.parquet",
    "partitionValues": {},
    "size": 13986213,
    "modificationTime": 1787054771843,
    "dataChange": true,
    "stats": "{\"numRecords\":200000,\"minValues\":{\"raw_json\":\"{\\\"model\\\": \\\"claude-haiku-4-5\\\", ...\",\"request_id\":\"00000966-edba-4a20-9aa3-7c8fedaf6901\",\"ts\":\"2026-04-01T00:00:00Z\"},\"maxValues\":{\"raw_json\":\"{\\\"model\\\": \\\"claude-sonnet-4-6\\\", ...\",\"ts\":\"2026-04-07T23:59:56Z\",\"request_id\":\"ffffca03-9af8-45b8-834e-439711aa7d05\"},\"nullCount\":{\"raw_json\":0,\"request_id\":0,\"ts\":0}}",
    "tags": null,
    "baseRowId": null,
    "defaultRowCommitVersion": null,
    "clusteringProvider": null
  }
}
```

**Key Info**:
- ✅ **operation**: "WRITE" (overwrite mode)
- ✅ **num_added_rows**: 200,000
- ✅ **num_added_files**: 1 Parquet file (Snappy compressed, 14 MB)
- ✅ **Schema**: 3 columns (request_id, ts, raw_json)
- ✅ **Stats**: min/max for filtering
- ✅ **Engine**: delta-rs (Rust implementation), Python binding

---

## Example 2: Silver Partitioned Table (WRITE with partitions)

**Path**: `_lakehouse/silver/llm_calls/_delta_log/00000000000000000000.json`

```json
{
  "commitInfo": {
    "timestamp": 1787054795432,
    "operation": "WRITE",
    "operationParameters": {
      "mode": "Overwrite"
    },
    "engineInfo": "delta-rs:py-1.6.2",
    "operationMetrics": {
      "execution_time_ms": 412,
      "num_added_files": 7,
      "num_added_rows": 190052,
      "num_partitions": 7,
      "num_removed_files": 0
    },
    "clientVersion": "delta-rs.py-1.6.2"
  }
}

{
  "protocol": {
    "minReaderVersion": 1,
    "minWriterVersion": 2
  }
}

{
  "metaData": {
    "id": "a1b2c3d4-e5f6-4g7h-8i9j-0k1l2m3n4o5p",
    "name": null,
    "description": null,
    "format": {
      "provider": "parquet",
      "options": {}
    },
    "schemaString": "{\"type\":\"struct\",\"fields\":[{\"name\":\"request_id\",\"type\":\"string\",\"nullable\":false,\"metadata\":{}},{\"name\":\"ts\",\"type\":\"timestamp\",\"nullable\":false,\"metadata\":{}},{\"name\":\"date\",\"type\":\"date\",\"nullable\":true,\"metadata\":{}},{\"name\":\"model\",\"type\":\"string\",\"nullable\":true,\"metadata\":{}},{\"name\":\"user_id\",\"type\":\"string\",\"nullable\":true,\"metadata\":{}},{\"name\":\"prompt_tokens\",\"type\":\"integer\",\"nullable\":true,\"metadata\":{}},{\"name\":\"completion_tokens\",\"type\":\"integer\",\"nullable\":true,\"metadata\":{}},{\"name\":\"latency_ms\",\"type\":\"integer\",\"nullable\":true,\"metadata\":{}},{\"name\":\"status\",\"type\":\"string\",\"nullable\":true,\"metadata\":{}}]}",
    "partitionColumns": ["date"],
    "createdTime": 1787054795300,
    "configuration": {}
  }
}

{
  "add": {
    "path": "date=2026-04-01/part-00000-abc123...snappy.parquet",
    "partitionValues": {
      "date": "2026-04-01"
    },
    "size": 2849372,
    "modificationTime": 1787054795400,
    "dataChange": true,
    "stats": "{\"numRecords\":27150,\"minValues\":{\"request_id\":\"00000000-0000-0000-0000-000000000000\",\"ts\":\"2026-04-01T00:00:00Z\",\"latency_ms\":50},\"maxValues\":{\"request_id\":\"ffffffff-ffff-ffff-ffff-ffffffffffff\",\"ts\":\"2026-04-01T23:59:59Z\",\"latency_ms\":5000},\"nullCount\":{\"status\":42}}"
  }
}

{
  "add": {
    "path": "date=2026-04-02/part-00000-def456...snappy.parquet",
    "partitionValues": {
      "date": "2026-04-02"
    },
    "size": 2741829,
    "modificationTime": 1787054795401,
    "dataChange": true,
    "stats": "{\"numRecords\":27143,...}"
  }
}

... (5 more files for dates 2026-04-03 through 2026-04-07)
```

**Key Info**:
- ✅ **num_added_files**: 7 (one per partition/date)
- ✅ **num_partitions**: 7 (date-based partitioning)
- ✅ **partitionColumns**: ["date"]
- ✅ **partitionValues**: Each file labeled with its date
- ✅ **num_added_rows**: 190,052 (after dedup from 200K)
- ✅ **Schema enhanced**: 9 columns (parsed from raw JSON)

---

## Example 3: Time Travel / MERGE (NB3)

**Path**: `_lakehouse/scratch/customers_tt/_delta_log/`

```
Version 0: 00000000000000000000.json
  operation: "WRITE"
  num_added_rows: 100000
  
Version 1: 00000000000000000001.json
  operation: "WRITE"  (schema evolution - add "tier" column)
  
Version 2: 00000000000000000002.json
  operation: "MERGE"
  operationMetrics: {
    "num_target_rows_matched": 50000,
    "num_target_rows_not_matched": 50000,
    "num_source_rows": 100000
  }
  
Version 3: 00000000000000000003.json
  operation: "WRITE"
  num_added_rows: 50  (bad data)
  
Version 4: 00000000000000000004.json
  operation: "RESTORE"
  operationParameters: {
    "version": 2
  }
  (Rolls back to v2, removes bad data from v3)
```

---

## Example 4: OPTIMIZE (NB2 / NB6)

**Path**: `_lakehouse/scratch/events_smallfiles/_delta_log/`

```
Version 0-199: 00000000000000000000.json ... 00000000000000000199.json
  operation: "WRITE"  or "APPEND"
  (200 small commits, 200 files)

Version 200: 00000000000000000200.json
  operation: "OPTIMIZE"
  operationMetrics: {
    "numFilesAdded": 10,
    "numFilesRemoved": 200,
    "totalConsideredFiles": 200,
    "totalFilesSkipped": 0
  }
  → Compacted 200 files → 10 files
  → Each "remove" entry tombstones old files (not deleted, marked)

Version 201: 00000000000000000201.json
  operation: "OPTIMIZE"
  operationMetrics: {
    "Z-ORDER": ["user_id"],
    "numFilesAdded": 10,
    "numFilesRemoved": 10
  }
  → Re-sort within files by user_id
  → Enables file pruning: all rows for user_id=4242 now in ~1 file
```

---

## Key Transaction Log Concepts

### 1. **commitInfo** - What happened?
- `timestamp`: Unix milliseconds when commit occurred
- `operation`: "WRITE", "APPEND", "MERGE", "RESTORE", "OPTIMIZE", "VACUUM", etc.
- `operationMetrics`: Specific numbers for that operation
- `clientVersion`: Which tool made the change (delta-rs, Spark, etc.)

### 2. **protocol** - Compatibility
- `minReaderVersion`: Reader must be at least this version
- `minWriterVersion`: Writer must be at least this version
- Ensures old clients can't corrupt tables they don't understand

### 3. **metaData** - Schema & structure
- `id`: Table UUID (stable across renames in newer Delta versions)
- `schemaString`: Current table schema (JSON-encoded Parquet schema)
- `partitionColumns`: List of partition columns
- `createdTime`: When the table was first created
- `configuration`: Custom table properties

### 4. **add** - Files created this version
- `path`: Relative path within table directory
- `partitionValues`: If partitioned, which partition this file belongs to
- `size`: File size in bytes (Parquet compressed)
- `modificationTime`: When file was written
- `dataChange`: true = real data, false = metadata-only files
- `stats`: Per-column min/max/null-count for file pruning

### 5. **remove** - Files deleted/tombstoned this version
- `path`: File being tombstoned
- `deletionTimestamp`: When tombstoned
- (File is NOT physically deleted immediately; VACUUM does that later)

---

## Log-Based Features Demonstrated

| Feature | Example | Benefit |
|---------|---------|---------|
| **Time Travel** | version=2 | Audit & rollback |
| **File Pruning** | stats (min/max) | Only read relevant files |
| **Partitioning** | date=2026-04-01 | Scan only needed partitions |
| **Schema Evolution** | medaData.schemaString | Add columns without rewrite |
| **Transaction Audit** | commitInfo history | Full lineage & debugging |
| **Garbage Collection** | remove + VACUUM | Reclaim space safely |

---

## File Structure Example

```
_lakehouse/gold/llm_daily_metrics/_delta_log/
├── 00000000000000000000.json   (v0: SQL aggregation - CREATE)
├── 00000000000000000001.json   (v1: Z-ORDER for dashboards)
├── 00000000000000000002.json   (v2: ADD more dates)
├── ...
└── _last_checkpoint            (Fast reader bootstrap)
    └── _delta_log.00000000002.checkpoint.parquet
```

Each line in a `.json` file is one JSON object:
- Some lines: commitInfo, protocol, metaData (setup info)
- Other lines: add/remove entries (file changes)
- Multiple lines = one logical version/commit

---

## Reading the Logs

```python
from deltalake import DeltaTable

dt = DeltaTable("_lakehouse/gold/llm_daily_metrics")

# Full history
for h in dt.history():
    print(f"v{h['version']} {h['operation']} {h.get('operationMetrics', {})}")

# Access specific version
dt_v2 = DeltaTable("_lakehouse/gold/llm_daily_metrics", version=2)
count = dt_v2.to_pyarrow_table().num_rows
```

This is what the notebooks do — they read the logs to **prove** dedup worked (Silver < Bronze), **prove** MERGE succeeded (history contains MERGE), **prove** partitions exist (file stats show partition values), etc.
