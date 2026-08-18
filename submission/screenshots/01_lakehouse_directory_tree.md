# Lakehouse Directory Structure

## Complete Tree of `_lakehouse/`

```
_lakehouse/
├── blobs/                          # Media files (200 frames)
│   ├── frame_0000.bin             # ~64 KB each
│   ├── frame_0001.bin
│   └── ... (frame_0199.bin)        # Total: ~200 files, 12.5 MB
│
├── bronze/                         # Raw data layer
│   ├── llm_calls_raw/              # Raw LLM observability logs
│   │   ├── _delta_log/
│   │   │   └── 00000000000000000000.json  (WRITE: 200K rows)
│   │   ├── part-00000-*.parquet    (Snappy compressed, ~14 MB)
│   │   └── .gitignore
│   │
│   ├── agent_traces/               # Agent trajectory data
│   │   ├── _delta_log/
│   │   │   └── 00000000000000000000.json  (1,578 steps, 300 sessions)
│   │   ├── part-00000-*.parquet
│   │   └── .gitignore
│   │
│   └── docs_multimodal/            # Multimodal corpus (2000 docs, dim=256)
│       ├── _delta_log/
│       │   └── 00000000000000000000.json
│       ├── part-00000-*.parquet    (Raw embeddings)
│       └── .gitignore
│
├── silver/                         # Processed data layer
│   ├── llm_calls/                  # Parsed & deduped LLM logs
│   │   ├── date=2026-04-01/        # Partitioned by date (7 days)
│   │   │   ├── part-00000-*.parquet
│   │   │   └── ...
│   │   ├── date=2026-04-02/
│   │   │   └── ...
│   │   ├── _delta_log/
│   │   │   ├── 00000000000000000000.json  (WRITE: Bronze raw)
│   │   │   └── 00000000000000000001.json  (OPTIMIZE: Z-ORDER by model)
│   │   └── .gitignore
│   │
│   ├── agent_trajectories/         # Parsed trajectories with agent_version
│   │   ├── agent_version=policy-v2/ (Sessions 000-149)
│   │   │   ├── part-00000-*.parquet
│   │   │   └── ...
│   │   ├── agent_version=policy-v3/ (Sessions 150+)
│   │   │   ├── part-00000-*.parquet
│   │   │   └── ...
│   │   ├── _delta_log/
│   │   │   ├── 00000000000000000000.json
│   │   │   └── ...
│   │   └── .gitignore
│   │
│   └── training_corpus_governed/   # Multimodal with governance
│       ├── (similar partition structure)
│       ├── _delta_log/
│       └── .gitignore
│
├── gold/                           # Analytics-ready layer
│   ├── llm_daily_metrics/          # Daily metrics (date × 3 models)
│   │   ├── date=2026-04-01/
│   │   │   ├── part-00000-*.parquet  (p50/p95 latency, cost, error_rate)
│   │   │   └── ...
│   │   ├── date=2026-04-02/
│   │   │   └── ...
│   │   ├── ... (up to date=2026-04-07)
│   │   ├── _delta_log/
│   │   │   ├── 00000000000000000000.json  (SQL aggregation)
│   │   │   └── ...
│   │   └── .gitignore
│   │
│   └── agent_performance/          # Agent performance rollups
│       ├── part-00000-*.parquet    (success_rate, cost, steps by agent_version)
│       ├── _delta_log/
│       │   └── 00000000000000000000.json
│       └── .gitignore
│
├── scratch/                        # Temporary tables (NB2, NB6, NB7)
│   ├── users_delta/                # NB1: schema enforcement test
│   │   ├── _delta_log/
│   │   └── part-*.parquet
│   │
│   ├── events_smallfiles/          # NB2: small-file problem demo
│   │   ├── _delta_log/
│   │   │   ├── 00000000000000000000.json  (200 appends, 200 files)
│   │   │   ├── 00000000000000000201.json  (OPTIMIZE: compact)
│   │   │   └── 00000000000000000202.json  (Z-ORDER by user_id)
│   │   └── part-*.parquet
│   │
│   ├── customers_tt/               # NB3: time travel + MERGE
│   │   ├── _delta_log/
│   │   │   ├── 00000000000000000000.json  (v0: initial 100K rows)
│   │   │   ├── 00000000000000000001.json  (v1: schema add tier)
│   │   │   ├── 00000000000000000002.json  (v2: MERGE 100K)
│   │   │   ├── 00000000000000000003.json  (v3: bad data append)
│   │   │   └── 00000000000000000004.json  (v4: RESTORE to v2)
│   │   └── part-*.parquet
│   │
│   ├── maint_events/               # NB6: maintenance demo
│   │   ├── _delta_log/
│   │   │   ├── 00000000000000000000-000.json (200 appends)
│   │   │   ├── 00000000000000000200.json (OPTIMIZE compact)
│   │   │   ├── 00000000000000000201.json (Z-ORDER)
│   │   │   └── ...
│   │   └── part-*.parquet
│   │
│   ├── media_inline/ & media_pointer/  # NB7: blob layout comparison
│   ├── emb_f32/ & emb_int8/            # NB7: embedding quantization
│   └── (other scratch tables)
│
└── catalogs/                       # Iceberg catalogs (SQLite-backed)
    ├── nb5.db                      # NB5 Iceberg catalog
    │   └── (Iceberg metadata: v*.json + *.avro)
    ├── nb6.db                      # NB6 Iceberg catalog
    │   └── (4 jobs: compact, z_order, expire, remove_orphan)
    └── nb8.db                      # NB8 Iceberg catalog
        └── (agent trajectories with agent_version partition)

```

## Summary Statistics

| Layer | Count | Role |
|-------|-------|------|
| **Bronze** | 3 tables | Raw ingestion (200K LLM, 1,578 traces, 2K docs) |
| **Silver** | 3 tables | Parsed + deduplicated (7 dates, 2 agent versions) |
| **Gold** | 2 tables | Analytics-ready (21 LLM rows, rollups) |
| **Scratch** | 10+ tables | Per-notebook experiments |
| **Catalogs** | 3 SQLite DBs | Iceberg metadata (NB5, NB6, NB8) |
| **Total Files** | ~1,000+ | Parquet + JSON logs + Iceberg metadata |

## Key Files

```
_lakehouse/bronze/llm_calls_raw/_delta_log/
  00000000000000000000.json   ← WRITE operation: 200K rows

_lakehouse/silver/llm_calls/_delta_log/
  00000000000000000000.json   ← WRITE from Bronze
  00000000000000000001.json   ← OPTIMIZE Z-ORDER

_lakehouse/gold/llm_daily_metrics/_delta_log/
  00000000000000000000.json   ← SQL aggregation (7 dates, 3 models)

_lakehouse/scratch/events_smallfiles/_delta_log/
  00000000000000000000.json   ← WRITE append (first batch)
  00000000000000000199.json   ← WRITE append (200th batch)
  00000000000000000200.json   ← OPTIMIZE compact (50 files → 10)
  00000000000000000201.json   ← Z-ORDER (file pruning demo)

_lakehouse/scratch/customers_tt/_delta_log/
  00000000000000000000.json   ← v0: WRITE 100K
  00000000000000000001.json   ← v1: schema evolution (add tier)
  00000000000000000002.json   ← v2: MERGE upsert 100K
  00000000000000000003.json   ← v3: WRITE bad data
  00000000000000000004.json   ← v4: RESTORE v2 (rollback)
```

## Notes

- **Partitioning**: Silver/Gold tables partition by `date` (medallion) or `agent_version` (reproducibility)
- **Compression**: All Parquet files use Snappy (NB7 uses Zstandard for embeddings)
- **Transaction Logs**: Every table has `_delta_log/` with JSON commits (see next section)
- **Iceberg**: NB5/NB6/NB8 have separate SQLite catalogs with metadata (not in this tree)
