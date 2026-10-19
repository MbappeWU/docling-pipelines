# Spec Delta

## Purpose

Provides vector database indexing and storage for chunked embeddings across supported vector store backends (OpenSearch, Milvus).

## ADDED Requirements

### Requirement: Automatic index or collection lifecycle management
The VectorDB operator SHALL automatically check whether the target index or collection exists before writing documents, creating it when absent and validating its schema when present, without exposing a manual `create_index` configuration flag.

#### Scenario: Index does not exist at transform time
- **WHEN** the operator transforms an input pyarrow Table and the configured index or collection does not exist
- **THEN** the operator creates the index or collection using detected embedding vector dimensions and feature mappings before inserting documents

#### Scenario: Index already exists at transform time
- **WHEN** the operator transforms an input pyarrow Table and the configured index or collection already exists
- **THEN** the operator validates the existing schema against input dimensions and feature mappings without recreating the index

#### Scenario: Creation failure marks documents as failed
- **WHEN** index creation fails due to backend errors or network failure
- **THEN** the transform returns the original table with node status `FAILED`, records failed documents with the error reason, and sets `processed_docs` to 0

### Requirement: Operator metadata contract
The VectorDB operator SHALL expose its configuration schema without any `create_index` parameter in operator introspection endpoints.

#### Scenario: Operator metadata retrieved
- **WHEN** `GET /api/v1/operators/metadata` is called
- **THEN** the metadata entry for `vectordb` does not contain `create_index` in its configurable parameters
