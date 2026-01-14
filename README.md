# EDGAR SEC Parser

A data ingestion and parsing system for transforming unstructured SEC EDGAR regulatory
filings (SGML / XBRL) into structured, queryable datasets for analytics and downstream
data pipelines.

This project focuses on correctness, reliability, and failure isolation when working
with heterogeneous financial documents where silent parsing errors can lead to
misleading or invalid conclusions.

---

## Context

SEC filings are semi-structured documents published in multiple formats, with frequent
inconsistencies, legacy edge cases, and malformed content. Naive parsing approaches
often fail silently, producing partial or incorrect data without clear signals.

This system was designed to ingest SEC filings and normalize their structure into a
relational model that can be safely used for analytics, research, and decision-support
pipelines, prioritizing traceability and reproducibility over raw throughput.

---

## Architecture

![Architecture Diagram](docs/arquitecture.png)

The architecture separates format detection, parsing, and normalization to explicitly
isolate upstream inconsistencies from downstream consumers.

At a high level, the system:
- Detects the document format (SGML, XBRL, or unknown)
- Routes documents to specialized parsers
- Normalizes extracted content into a stable schema
- Persists metadata and extracted facts to PostgreSQL

This separation allows failures and parser-specific issues to be handled without
corrupting the structured data layer.

---

## Data Flow

1. Discover and fetch SEC EDGAR filings
2. Detect document format and structural characteristics
3. Route the document to the appropriate parser (SGML, XBRL, or fallback)
4. Extract and normalize relevant metadata and financial facts
5. Persist structured data to PostgreSQL for querying and downstream use

Processing is batch-oriented to favor reproducibility, auditability, and safe
reprocessing over low-latency ingestion.

---

## Key Design Decisions & Trade-offs

- **Specialized parsers vs a single universal parser**  
  Using dedicated SGML and XBRL parsers increases integration complexity but significantly
  improves correctness across heterogeneous document formats.

- **Normalization layer as a hard boundary**  
  All parser outputs pass through a normalization layer to enforce schema consistency
  and prevent malformed data from leaking into storage.

- **Batch processing over streaming**  
  Batch ingestion simplifies error handling, backfills, and reprocessing at the cost of
  higher end-to-end latency.

- **Relational storage (PostgreSQL)**  
  PostgreSQL was chosen to enable explicit schemas, constraints, and SQL-based analytics,
  trading some flexibility for stronger guarantees.

---

## Assumptions

- SEC filings may be incomplete or malformed
- Schema drift is expected across time and filing types
- Correctness and traceability are more important than ingestion speed
- Reprocessing and backfills must be supported as first-class operations

---

## Failure Scenarios & Handling

The system explicitly accounts for:
- Malformed or partially corrupted SGML/XBRL documents
- Unexpected document structures
- Parser-specific failures
- Inconsistent or missing sections within filings

Failures are logged with sufficient metadata to allow targeted reprocessing without
data loss. When possible, fallback parsing strategies are applied to recover partial
content safely.

---

## Testing

The project includes:
- Unit tests for parser integrations
- Integration tests for ingestion and persistence
- Basic performance tests to detect regressions

Testing prioritizes correctness, schema consistency, and failure handling rather than
absolute throughput benchmarks.

---

## Project Structure
```
sec_extractor/
├── parsers/      # SGML / XBRL parser integrations
├── core/         # Routing and processing logic
├── discovery/    # Filing discovery utilities
├── storage/      # Database models and persistence
└── tests/        # Unit, integration, and performance tests

docs/
└── arquitecture.png

scripts/

notebooks/
```

---

## What This Project Demonstrates

- Designing ingestion pipelines for unreliable external data sources
- Isolating third-party parsing complexity behind stable interfaces
- Making explicit trade-offs between correctness, complexity, and performance
- Treating data quality and silent failures as system-level concerns

---

## Future Improvements

- Incremental ingestion and change detection
- Schema versioning and explicit data contracts
- Observability metrics for ingestion health and failure rates
- Integration into a larger orchestration framework (e.g., Airflow)

---

Built as a practical exercise in data ingestion, reliability, and system design for
decision-critical financial data.
