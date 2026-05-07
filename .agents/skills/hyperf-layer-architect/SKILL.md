---
name: hyperf-layer-architect
description: Guide Hyperf PHP project architecture by layered responsibility, static analysis, and memory analysis rules. Use when Codex needs to add, modify, refactor, or review Hyperf code involving Http controllers, requests, middleware, jobs, commands, processes, listeners, services, repositories, models, exceptions, schemas, support objects, factories, contracts, strategies, dispatchers, PHPStan/static analysis, or Hyperf/Swoole long-running memory risk, and must decide where logic belongs without crossing layer boundaries.
---

# Hyperf Layer Architect

## Overview

Use this skill to place Hyperf code according to clear layer responsibilities. Keep entry layers thin, keep business orchestration in services, keep data access in repositories, move stable support logic out of oversized services, and verify changes with static analysis and memory-risk checks.

## Workflow

1. Inspect existing project conventions before editing.
2. Identify whether the requested code is an entry point, business flow, data access, model structure, documentation structure, support object, factory, contract, strategy, or dispatcher.
3. Put new code in the narrowest layer that owns the responsibility.
4. Keep entry layers focused on receiving input, validating, authorizing, forwarding to services, and returning output.
5. Keep services focused on use cases and business orchestration.
6. Extract stable assembling, parsing, mapping, formatting, resolving, or building logic into `Support/` instead of growing large service classes.
7. Run project static analysis after meaningful PHP changes.
8. Perform memory analysis when touching Hyperf long-running paths, queues, processes, listeners, websocket handlers, timers, caches, large collections, static state, or coroutine context.
9. After code changes, summarize which layer each changed file belongs to and why.

## Layer Responsibilities

| Directory | Role | Responsibilities | Put Here | Do Not Put Here |
|---|---|---|---|---|
| `Http/` | Request entry layer | Receive HTTP requests, validate parameters, control permissions, return responses | `Controller`, `Request`, `Middleware`, Swagger-related definitions | Business flow, database operations, third-party call details |
| `Job/` | Queue entry layer | Receive queue payloads, trigger business services, control retry and timeout behavior | Async queue jobs | Large business logic, reflection calls to `protected` or `private` methods |
| `Command/` | CLI entry layer | Trigger commands and scheduling entry points | Commands | Complex business implementation |
| `Process/` | Long-running process entry layer | Queue consumers, daemons, long-running process entry points | Consumers, long-running processes | Main business flow implementation |
| `Listener/` | Event entry layer | Respond to events and delegate to services | Listeners, event subscription entry points | Complex business orchestration |
| `Service/` | Business service layer | Business orchestration, business capabilities, use case implementation | Application services, business sub-services | Mixed large classes, long-accumulated assembling or parsing details |
| `Repository/` | Data access layer | Querying, persistence, aggregate reads and writes | Repositories, query wrappers | Notifications, external APIs, business flow |
| `Model/` | Data model layer | Data structure, relationships, casts, enum associations | Models, casts, enum associations | Complex business rules, flow control |
| `Exception/` | Exception layer | Unified exception expression and handling | Business exceptions, exception handlers | Normal business flow |
| `Schema/` | Documentation structure layer | API documentation and Swagger structures | Schemas, Swagger definitions | Business logic |
| `Support/` | Support object layer | Stable objects that should not keep accumulating inside services | `Builder`, `Resolver`, `Assembler`, `Normalizer`, `Formatter`, `Mapper`, `Parser` | Complete business flow, database reads or writes |
| `Support/Factory/` | Factory layer | Object creation and instantiation dispatch | `Factory`, `Creator`, `Instantiator` | Main business flow |
| `Contracts/` | Contract layer | Interfaces and abstract capability boundaries | Interfaces, contracts | Concrete implementations |
| `Strategy/` | Strategy layer | Multi-branch rules and switching between implementations | `Strategy`, `Policy`, `Rule`, `Matcher`, `Selector` | Large process orchestration |
| `Dispatcher/` | Dispatch layer | Dispatch tasks, notifications, or events to different targets | `Dispatcher`, `Publisher`, `Router`, `Trigger` | Complete business decisions and main flow |

## Placement Rules

- Put HTTP request receiving, validation, authorization, and response formatting in `Http/`.
- Put queue payload handling, retry configuration, timeout configuration, and service invocation in `Job/`.
- Put CLI command input and scheduling triggers in `Command/`.
- Put long-running process or consumer entry points in `Process/`.
- Put event reaction and service delegation in `Listener/`.
- Put use cases, business orchestration, and business capability composition in `Service/`.
- Put queries, persistence, and aggregate reads or writes in `Repository/`.
- Put data structures, relationships, casts, and enum associations in `Model/`.
- Put business exceptions and exception handlers in `Exception/`.
- Put Swagger and API documentation structures in `Schema/`.
- Put stable assembling, resolving, normalizing, formatting, mapping, parsing, or building logic in `Support/`.
- Put object construction and instantiation selection in `Support/Factory/`.
- Put interfaces and abstract boundaries in `Contracts/`.
- Put interchangeable rules, policies, matchers, selectors, or strategies in `Strategy/`.
- Put target selection and delivery dispatch for tasks, notifications, or events in `Dispatcher/`.

## Boundary Rules

- Do not put business flow in controllers, jobs, commands, processes, or listeners.
- Do not put database operations or external API call details in `Http/`.
- Do not put notifications, external API calls, or business orchestration in repositories.
- Do not put complex business rules or process control in models.
- Do not use exceptions as a normal business process mechanism.
- Do not put business logic in schemas or Swagger definitions.
- Do not put complete business flows or database reads and writes in `Support/`.
- Do not put concrete implementations in `Contracts/`.
- Do not let strategies become large business orchestration classes.
- Do not let dispatchers own full business decisions or main flows.

## Refactoring Guidance

When a class becomes too large, identify the type of responsibility causing the growth:

- Move request-specific validation to `Http/Request`.
- Move repeated query logic to `Repository/`.
- Move stable data conversion to `Support/Mapper` or `Support/Normalizer`.
- Move response or payload assembling to `Support/Assembler`.
- Move parsing details to `Support/Parser`.
- Move formatting details to `Support/Formatter`.
- Move object construction logic to `Support/Factory`.
- Move multi-implementation selection to `Strategy/`.
- Move delivery target routing to `Dispatcher/`.
- Move interface boundaries to `Contracts/`.

## Review Checklist

When reviewing Hyperf code, check:

- Are entry layers thin?
- Is the main business flow in `Service/`?
- Are database reads and writes isolated in `Repository/`?
- Are models limited to data structure, relationships, casts, and enum associations?
- Is repeated assembling, parsing, mapping, formatting, or resolving extracted to `Support/`?
- Are interfaces separated into `Contracts/`?
- Are multi-branch rules represented as strategies instead of large `if/else` blocks in services?
- Are dispatch decisions separated from the actual business flow?

## Static Analysis

Run static analysis for non-trivial PHP changes:

1. Inspect `composer.json` scripts first.
2. Prefer the project script when available:

```bash
composer analyse
```

3. If the Composer script is unavailable, use the local PHPStan binary and project config:

```bash
vendor/bin/phpstan analyse --memory-limit 300M
```

4. If the repository has no PHPStan config, analyze the main Hyperf source paths directly:

```bash
vendor/bin/phpstan analyse app config --memory-limit 300M
```

5. For very small changes, also run syntax checks on changed PHP files when useful:

```bash
php -l path/to/ChangedFile.php
```

Treat static analysis findings as design feedback, not just syntax feedback. Common layer-related findings include missing return types, mixed arrays crossing layers, repositories returning unstable shapes, controllers doing too much normalization, and services hiding parsing or mapping details that should move to `Support/`.

## Memory Analysis

Hyperf runs on Swoole and commonly has long-running workers. Memory issues often come from retained state rather than one slow request.

Perform memory analysis when changing:

- `Process/`, queue consumers, jobs, listeners, websocket handlers, timers, or long loops.
- Static properties, singleton services, global registries, in-memory caches, or closures captured by long-lived objects.
- Coroutine context, request-scoped data, large arrays, generators, streams, file handles, Redis or database result sets.
- Batch processing, retry loops, event dispatch, notification dispatch, or third-party API fan-out.

Check for these risks:

- Mutable request, user, room, connection, or payload data stored in static properties or singleton services.
- Large arrays retained after processing instead of being streamed, chunked, or released.
- Closures retaining `$this`, request objects, large payloads, connection objects, or container instances.
- Coroutine-local data not cleared after work completes.
- Timers, listeners, channels, callbacks, or subscriptions registered repeatedly.
- Repository queries loading unbounded data instead of paging, chunking, or selecting required fields.
- Long-running loops without cleanup, backoff, timeout, or stop conditions.

Use the project's available tooling before inventing commands:

1. Check available commands:

```bash
php bin/hyperf.php list
```

2. If memory-related Hyperf commands or project scripts exist, use them and report the exact command.
3. If no memory command exists, use focused instrumentation around the changed path: record `memory_get_usage(true)` and `memory_get_peak_usage(true)` before, during, and after repeated execution.
4. For queue, process, websocket, or listener changes, prefer repeated execution in a bounded loop over a single call. Compare memory before and after the loop.
5. Release large local variables with normal scoping first; use `unset()` only for clearly large retained values. Use `gc_collect_cycles()` only after references are released and only as a diagnostic or explicit cleanup step.

Memory analysis does not require every feature change to include a benchmark. It is required when the changed code can live beyond a single HTTP request or can repeatedly process large payloads.

## Response Expectations

When using this skill for implementation or review, report the layer decision briefly:

- Changed files and their intended layer.
- Why each file belongs in that layer.
- Any boundary risks that remain.
- Static analysis command and result, or why it was not run.
- Memory analysis performed for long-running or high-volume paths, or why it was not needed.
