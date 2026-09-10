# HiQ DBOS SQLite candidate

This downstream fork integrates community SQLite support; it is not an official DBOS SQLite release. The upstream MIT license and contribution history are retained.

## Source policy

- Stable SDK base: upstream `release/v4.27` / `v4.27`, commit `ef3534036901fa4f54d5eb6c4596e34fcf7d939f`.
- SQLite contribution: [upstream PR #1288](https://github.com/dbos-inc/dbos-transact-ts/pull/1288), commit `80d0776e1ee8b1e210cbfbcd6fbe564e655b5522`.
- Integration branch: `feat/sqlite-stable`, targeting this fork's `release/v4.27`.
- The earlier `feat/sqlite-system-database` / PR #1 follows upstream `4.28-preview` and is research evidence only. Its database files must not be reused with this stable candidate.

Use stable upstream releases for adoption. Do not expand this fork to follow unreleased PostgreSQL features. Reevaluate the fork when upstream ships supported SQLite persistence.

## Runtime and data boundaries

Select SQLite explicitly with `systemDatabaseUrl: 'sqlite:////absolute/path/system.sqlite'`. PostgreSQL remains the default regardless of Node version. SQLite needs Node.js 22.13 or newer; PostgreSQL continues to support Node.js 20.

The contributed backend uses `node:sqlite` DatabaseSync, one serialized connection, WAL, polling and a PostgreSQL-to-SQLite SQL translation layer. It is not a general PostgreSQL implementation or a public backend SPI. PostgreSQL caller-owned transaction APIs (`pg.ClientBase`), LISTEN/NOTIFY and JSONB containment are not SQLite guarantees.

SQLite migrations are explicitly versioned: the contributed baseline is 107; the stable integration adds migration 108 for partition limits. They must not automatically follow the number of PostgreSQL migrations. `runMigrations: false` checks an existing database without creating it. Databases from an experimental preview build are not migration inputs.

Synchronous SQLite execution can block its Node event loop. A desktop consumer must validate its actual Electron/Node runtime and execute outside the UI/main process. Cancellation of a workflow does not by itself stop an executing external operation.

Go DBOS and TypeScript DBOS histories are not interchangeable. Do not point this SDK at an existing Go case database. Preserve canonical case data and either drain existing runs or explicitly design and verify migration.

## Validation and consumption

Validation is in progress. No application engine has been replaced. Build, lint, package checks, SQLite and PostgreSQL tests, real process recovery and target-runtime checks are separate gates. A successful merge is not application acceptance.

Local evidence on 2026-09-10:

- Build, ESLint and package export/type checks pass.
- SQLite: 21 dedicated tests plus 293 tests across core workflows, management, client, application scoping, recovery, patching, streams, serialization, outcome ownership, batch enqueue, attributes and versions pass. Backend-specific skips are retained; this is not the complete upstream suite.
- PostgreSQL: targeted workflow, queue, client and management suites pass after fixing a timestamp-tie-dependent pagination fixture. No production timestamp changes or artificial delays are used.
- Electron 40.8.5 / Node 24.14.0, run as a separate test process, loads `node:sqlite` and completes a real SDK workflow. This does not verify a packaged desktop application or Windows.
- The first CI matrix passed PostgreSQL on Node 20/22/24 and package tests, but failed SQLite. Follow-up fixes add an explicit COUNT alias for queue admission, correct PostgreSQL application-database URLs in mixed-backend tests, and allow equal millisecond timestamps. Targeted SQLite 21/21 and PostgreSQL 22/22 checks pass; the new full CI matrix is pending.
- File connections explicitly set WAL and synchronous=FULL rather than inheriting host SQLite compile defaults. Reopen tests verify both settings; this is not a simulated power-loss test.
- The follow-up CI passed the full Node 24 SQLite suite. Node 22 exposed a stream test assuming notification latency despite SQLite's default one-second polling. The test now explicitly requests 50ms SQLite polling without relaxing its 500ms bound; PostgreSQL still uses notifications, and default polling has separate coverage. Node 22 streaming regression: 37 passed, 5 existing backend-specific skips. Full final matrix remains pending.

Upstream namespace publication is disabled for forks. Consume only an immutable verified downstream artifact, not an unbuilt branch or a mutable replacement for the upstream npm package. No automatic desktop release is authorized by this integration.

Keep application-specific experts, prompts and customer materials out of this public SDK fork.
