# EDGAR SEC Parser

A data ingestion and parsing system for transforming unstructured SEC regulatory filings
(SGML / XBRL) into structured, queryable datasets for analytics and downstream processing.

This project focuses on reliability, correctness, and failure handling when working with
heterogeneous financial documents that are inconsistent by nature.

## Context

SEC filings are semi-structured documents with multiple formats, frequent inconsistencies,
and legacy edge cases. Manual processing or naive parsing approaches often fail silently,
producing partial or misleading data.

This system was designed to ingest SEC filings and normalize their structure into a
relational model that can be safely used for analytics, research, and downstream pipelines.

## Architecture

High-level flow:

SEC Filings (SGML / XBRL)
→ Format detection
→ Parser selection (SGML / XBRL / integrated)
→ Normalization layer
→ PostgreSQL storage (metadata + extracted facts)

The system uses specialized SEC parsing libraries (`secsgml`, `secxbrl`) wrapped behind a
common interface to isolate parser-specific complexity from the ingestion logic.

## Data Flow

1. Discover and fetch SEC filings
2. Detect document format and structure
3. Route to the appropriate parser
4. Normalize extracted content into a stable schema
5. Persist structured data and metadata to PostgreSQL

Batch-oriented processing is used to favor reproducibility and traceability over raw
throughput.

## Key Design Decisions & Trade-offs

- **Multiple parsers vs single universal parser**  
  Using specialized parsers increases complexity but significantly improves correctness
  across heterogeneous document formats.

- **Relational storage vs document storage**  
  PostgreSQL was chosen to enable explicit schemas, constraints, and downstream SQL-based
  analytics at the cost of some ingestion flexibility.

- **Batch processing over streaming**  
  Filings are processed in batches to simplify error recovery, reprocessing, and auditing.

## Assumptions

- SEC filings may be malformed or incomplete
- Schema drift is expected across filings and time
- Throughput requirements are secondary to correctness
- Reprocessing and backfills must be supported

## Failure Scenarios & Handling

The system explicitly handles:
- Malformed SGML / XBRL documents
- Partial filings
- Parser-specific failures
- Unexpected document structures

Failures are logged with sufficient metadata to allow reprocessing without data loss.
Fallback parsing strategies are applied where possible.

## Testing

The project includes:
- Unit tests for parser integration
- Integration tests for ingestion and storage
- Basic performance tests to detect regressions

Testing focuses on correctness and failure handling rather than absolute performance.

## Project Structure

sec_extractor/
├── parsers/ # Parser integrations (SGML / XBRL)
├── core/ # Routing and processing logic
├── discovery/ # Filing discovery
├── storage/ # Database models and persistence
├── tests/ # Unit, integration, and performance tests

yaml
Copy code

## What This Project Demonstrates

- Designing ingestion pipelines for unreliable external data
- Isolating third-party parsing complexity
- Making explicit trade-offs between correctness and performance
- Treating data quality and silent failures as system-level concerns

## Future Improvements

- Incremental ingestion and change detection
- Schema versioning and data contracts
- Observability metrics for ingestion health
- Integration into a larger orchestration framework (e.g., Airflow)

---

Built as a practical exercise in data ingestion, reliability, and system design for
decision-critical financial data.
