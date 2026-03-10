# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**lakeFS** is a data version control system that provides Git-like versioning for data lakes. It transforms object storage (S3, Azure, GCS) into versioned repositories, enabling branching, committing, and merging of data at scale.

## Build and Development Commands

### Main Build (Go Backend)
```bash
make build              # Build lakefs and lakectl binaries
make gen-api            # Generate API code from OpenAPI spec
make gen-code           # Run all code generators (protobuf, mocks, etc.)
make lint               # Lint Go and UI code
make clean              # Clean build artifacts
```

### Testing
```bash
make test               # Run all tests (Go + HadoopFS)
make test-go            # Run Go tests with coverage
make test-hadoopfs      # Run HadoopFS tests (Maven)
make run-test           # Run tests without regenerating code (faster)
make fast-test          # Run tests without race detector
make esti               # Run end-to-system integration tests
make system-tests       # Run system tests locally
```

### Running Single Tests
```bash
# Run a specific Go test
go test -v -run TestSpecificName ./path/to/package

# Run tests in a specific package
go test -v ./pkg/graveler

# Run tests with race detector (recommended)
go test -race ./path/to/package
```

### Frontend Development (webui/)
```bash
cd webui
npm run dev             # Development server with hot reload
npm run build           # Production build
npm run test            # Run frontend unit tests
npm run test-watch      # Watch mode for tests
npm run lint            # Lint TypeScript/React code
```

### Client SDK Generation
```bash
make clients            # Generate all client SDKs (Python, Java, Rust)
make client-python      # Generate Python SDK
make client-java        # Generate Java SDK
make sdk-rust           # Generate Rust SDK
```

### Docker
```bash
make build-docker       # Build Docker image
```

## Architecture

### Core Components

**Graveler** (`pkg/graveler/`): The version control engine. Similar to Git's object model, it manages:
- Commits, branches, tags, and merges
- Staging area for uncommitted changes
- Reference management (branch pointers, commit ancestry)
- Tree metadata operations (listing, diffing)
- Protobuf-based storage model

**Block Storage Layer** (`pkg/block/`): Abstraction over underlying object storage:
- Supports AWS S3, Azure Blob Storage, Google Cloud Storage, and local filesystem
- Adapts block storage to a key-value store interface
- Physical address translation from logical addresses

**Catalog** (`pkg/catalog/`): Object metadata and namespace management:
- Maps repository + ref + path to physical storage addresses
- Manages Iceberg table metadata integration
- Entry operations (get, list, create, delete)

**Authentication & Authorization** (`pkg/auth/`, `pkg/authentication/`):
- Multiple auth methods: JWT, Basic Auth, OIDC, SAML
- Role-based access control (RBAC)
- Repository and prefix-level permissions
- External identity provider integration

**S3 Gateway** (`pkg/gateway/`): S3-compatible API implementation:
- Translates S3 API calls to lakeFS operations
- Handles multipart uploads, versioning, and metadata
- Enables seamless integration with S3-compatible tools

**API Layer** (`pkg/api/`): HTTP API built on Chi router:
- RESTful endpoints for repositories, branches, commits, objects
- OpenAPI v3 specification (`api/swagger.yml`)
- Middleware for auth, metrics, audit logging
- ETag-based caching and conditional requests

**Actions Engine** (`pkg/actions/`): Lua scripting for data processing:
- Pre-commit and post-commit hooks
- Custom data validation and transformation
- Integration with external systems via webhooks

### Key Architectural Patterns

**Write-Audit-Publish**: Data changes go through isolated branches, can be validated, then merged to production.

**Copy-on-Write**: Branches don't copy data—they're pointers to commits. Only changed data is stored.

**Modular Services** (`modules/`): Independent service modules for API, auth, authentication, block storage, catalog, config, gateway, and license.

**Generated Code**: Much of the boilerplate is generated:
- API handlers from OpenAPI spec (`go:generate` directives in `pkg/api/apigen/`)
- Protocol buffers for inter-service communication
- Mock interfaces for testing

**KV Store Abstraction** (`pkg/kv/`): Key-value interface over:
- CockroachDB Pebble (LSM tree storage engine) for local storage
- Postgres for distributed deployments
- In-memory stores for testing

### Frontend Architecture (webui/)

**React SPA** built with:
- Vite for fast development and optimized builds
- Material-UI v5 for component library
- React Router v6 for navigation
- Monaco Editor for code editing
- DuckDB WebAssembly for in-browser data queries

**State Management**: React hooks and context, minimal external state management.

**API Client**: Generated from OpenAPI spec with TypeScript types.

### Technology Stack

**Backend**: Go 1.25.5, Chi router, Protocol Buffers, Pebble KV store
**Frontend**: React 18.2, TypeScript, Vite, Material-UI
**Storage**: CockroachDB Pebble, PostgreSQL (optional)
**Metrics**: Prometheus
**Testing**: Go testing, Vitest (frontend), Playwright (E2E)

## Important Development Notes

**Code Generation**: After modifying certain files (especially API definitions, protobuf files), you must run `make gen-api` and/or `make gen-code`. CI checks validate that generated code is up-to-date.

**Validation Targets**: The Makefile includes `validate-*` targets that check if generated code matches source (e.g., `validate-proto`, `validate-api`).

**Hadoop/Spark Integration**: Java client code uses Maven for testing (`make test-hadoopfs`).

**Documentation**: Docs are in `docs/` directory using MkDocs. Serve locally with `make docs-serve`.

**Configuration**: Supports multiple configuration modes—quickstart (local dev), local defaults, or full YAML config at `~/.lakefs.yaml`.
