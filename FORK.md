# HiQ SQLite integration candidate

This is a downstream integration fork, not an official DBOS SQLite release.
The upstream MIT license and contribution history are retained.

## Source and scope

- Upstream: `dbos-inc/dbos-transact-ts`, base `be82b1f4ace6210ab02473e92c0add1464495b84`.
- SQLite contribution: upstream PR [#1288](https://github.com/dbos-inc/dbos-transact-ts/pull/1288), head `80d0776e1ee8b1e210cbfbcd6fbe564e655b5522`.
- Integration branch: `feat/sqlite-system-database`.
- Purpose: evaluate a TypeScript + SQLite durable engine for local desktop execution without a PostgreSQL service.

SQLite is explicitly selected using `systemDatabaseUrl: 'sqlite:////absolute/path/system.sqlite'`.
The PostgreSQL default is retained regardless of Node.js version: upgrading Node must not silently switch the persistence backend.
SQLite requires Node.js 22.13 or newer. The PostgreSQL backend continues to support Node.js 20.

## Constraints

- The contributed backend uses `node:sqlite` DatabaseSync, one serialized connection, WAL and a PostgreSQL-to-SQLite SQL translation layer. It is not a general PostgreSQL-compatible database or a public database adapter API.
- Synchronous SQLite calls can block the Node event loop. Desktop integration must isolate execution from the UI/main process and test cancellation and process recovery.
- PostgreSQL LISTEN/NOTIFY and PostgreSQL-specific query timeout enforcement are not SQLite capabilities. SQLite notifications use polling; its lock busy timeout is not a statement execution timeout.
- A TypeScript SQLite database is not interchangeable with a Go DBOS database. Existing workflow histories must not be opened as if they belonged to this SDK. Migration must separately account for in-flight runs and canonical application data.
- No automatic npm publication from this fork under the upstream `@dbos-inc` namespace. Consumers must use a verified, immutable downstream build; an unbuilt Git branch is not a deployable package.

## Validation and adoption

Integration is in progress. Passing a smoke test or merging this branch does not establish production readiness.

Verified locally on Node.js 24.19.0 (macOS): build including workspace packages, ESLint, package exports/type resolution, SQLite schema upgrade from the contributed version 107, core workflow execution, portable serialization, outcome ownership, patching, and generic client operations. PostgreSQL-only transaction and statement-timeout tests are explicitly separated from SQLite; they remain in the PostgreSQL test run.

Current fixes beyond the contributed PR include explicit SQLite migrations 108–113, lazy database opening for `runMigrations: false`, latest queue dispatch API/limits, retention/payload tables, and removal of artificial checkpoint clock sleeps/timestamps. The test suite declares its `ts-node` subprocess dependency instead of relying on incidental workspace dependencies.

Broader recovery, queue and regression runs are still pending. No desktop engine migration has been performed.
Required evidence includes SQLite SDK tests, PostgreSQL regression tests, packaging, process-kill recovery, durable human-input waits, parallel steps, cancellation, and restart/version behavior on the target desktop runtime.
Only verified capabilities should be enabled by consumers; retain the existing engine until the application migration gates pass.

Upstream remains the source of general SDK documentation. Keep downstream changes limited to SQLite integration and demonstrated compatibility fixes; do not add application-specific expert behavior here.
