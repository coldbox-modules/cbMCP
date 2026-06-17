# Project Guidelines

## Purpose

This repository is the **ColdBox MCP Server Module** (`cbMCP`), a ColdBox module that exposes your running ColdBox application as an **MCP (Model Context Protocol) server**.

It extends the `bx-ai` module's MCP infrastructure to provide live, read-only introspection tools for the entire ColdBox ecosystem — routing, handlers, modules, WireBox, CacheBox, LogBox, schedulers, async executors, and more. External AI clients (Claude Desktop, VS Code Copilot, Cursor, etc.) can connect via HTTP/SSE transport to query and monitor the runtime.

Key capabilities include:

- **System & Runtime** — Engine detection (BoxLang/Lucee/Adobe), framework settings, application structure, setting inspection, and app reinit
- **Handlers** — All registered event handlers with their public actions (core and per-module)
- **Routing** — Application routes, module routing tables, and router configuration (base URL, multi-domain, loose matching)
- **Modules** — Full module registry with activation status, versions, paths, settings, and conventions
- **WireBox** — DI container mappings with scope, autowire, alias, delegates, thread safety, and virtual inheritance
- **CacheBox** — All cache providers with hit/miss/eviction statistics, key inspection, and cache clearing operations
- **LogBox** — Logger registry, appenders, log levels, root logger, and last-error detection from file-based appenders
- **Interceptors** — All registered interception points and their listener pools
- **Schedulers** — Task schedulers (global + module), task execution stats, and task lifecycle (run/pause/resume)
- **Async** — Executor registry with thread pool info and task statistics
- **Resources** — Ambient read-only context (`coldbox://app/settings`, `coldbox://app/modules`, `coldbox://app/routes`, `coldbox://app/handlers`)
- **Prompts** — Pre-packaged AI workflows: `coldbox_app_overview`, `debug_handler`, `cache_health_report`, `interceptor_audit`

## Architecture

- Module metadata and runtime lifecycle live in `ModuleConfig.bx`.
- BoxLang source belongs under `models/`; the test harness lives in `test-harness/`.
- Tool classes live in `models/tools/` and are auto-discovered via `this.scan("cbMCP.models.tools")` in the `onLoad()` lifecycle method.
- The MCP server is registered programmatically in `ModuleConfig.bx` via `mcpServer()`, with resources, prompts, and tool scanning chained onto it.
- The public web endpoint is at `/{entryPoint}/` (default: `/cbmcp`), routed through `config/Router.bx` via `.toMCP("cbMCP")`.
- Build artifacts live in `build/Build.cfc` and produce a zip distribution for ForgeBox.

## Build And Packaging

- Use CommandBox (`box`) for build tasks via the scripts defined in `box.json`.
- Key scripts:
  - `build:module` — Build module zip: `box run-script build:module`
  - `build:docs` — Generate DocBox API docs: `box run-script build:docs`
  - `release` — Full release recipe: `box run-script release`
  - `install:dependencies` — Install dependencies for both module and test harness: `box run-script install:dependencies`
- Prefer narrow validation first: run tests via the test harness or execute `box testbox run` against the test runner.
- The build pipeline (`build/Build.cfc`) handles: test execution, source assembly, doc generation, checksum creation, and zip packaging.
- Source excludes (`build/Build.cfc`) exclude `build/`, `test-harness/`, `node-modules/`, dot-files, `server-*.json`, and docker config from the final binary.

## Conventions

- Follow `.editorconfig` indentation and line-ending rules. This repo uses **tabs** by default, with **spaces (2)** for YAML and markdown.
- Follow the project's `.bxformat.json` formatting (tabs, 4-space indent, aligned assignments/properties, bracket padding, semicolons).
- Keep ColdBox module metadata in `box.json`, build scripts in `build/Build.cfc`, and the module registration in `ModuleConfig.bx` aligned when changing names, versions, or identifiers.
- Only use acronyms when they are necessary and represent well-known constructs; prefer full descriptive words in names and documentation.
- Tool classes extend no base — they are plain BoxLang classes annotated with `@mcpTool` and `@AITool` on their public methods.
- The `@mcpTool` annotation must appear before `@AITool` on every tool method.

## Tool Naming Conventions

All MCP/AI tool methods across the module must follow consistent naming patterns. The conventions below are **mandatory** for new tools and must be maintained when refactoring existing ones.

### Naming Style

Every tool method uses `<verb>_<noun>` format in `snake_case`, reflecting what the tool does:

| Pattern | Example |
|---|---|
| `get_*` | `get_caches()`, `get_schedulers()`, `get_handlers()` |
| `list_*` | `list_routes()`, `list_module_routes()` |
| `has_*` | `has_mapping()` |
| `cache_*` | `cache_key_exists()` |
| `clear_*` | `clear_all()`, `clear_item()`, `clear_event()` |
| `time_*` | `time_now()` |
| `reinit_*` | `reinit_app()` |
| `read_*` | `read_log_entries()` |
| `run_*` / `pause_*` / `resume_*` | `run_task()`, `pause_task()`, `resume_task()` |

### Standard Method Patterns

Every tool class **should** implement the following standard methods where applicable:

| Pattern | Signature | Purpose |
|---|---|---|
| `get_<plural>()` | Returns all entities | Holistic view of the subsystem |
| `get_<names>` | Returns array of names only | Lightweight listing for dropdowns/selectors |
| `get_<singular>( name )` | Returns a single entity by name | Drill-down into one item |
| `has_<singular>( name )` | Returns boolean | Existence check |

Methods prefixed with an underscore (e.g. `_buildHandlerList`, `_resolveTask`) are private helpers and must **not** be annotated with `@mcpTool` or `@AITool`.

### Annotation Order

Always declare annotations in this order:

```boxlang
@mcpTool
@AITool
public struct function get_caches() {
```

`@mcpTool` first, then `@AITool` — consistent across every tool method in every file. Never reverse these.

### Error Handling

- Tool methods **must never throw exceptions**. Always return error structs:

  ```boxlang
  { "error": true, "message": "No scheduler found with the name: myScheduler. Use get_schedulers() to see available schedulers." }
  ```

- Error messages must include the user's input and guide them toward the corrective action (which tool to call instead).

### Method Documentation

Every `@mcpTool` / `@AITool` method must have a preceding javadoc comment block that:

1. Describes what the tool does in one sentence
2. Lists each `@param` with a type hint in the description
3. Mentions related tools when an error struct would reference them

### File Header

Every tool file starts with the standard copyright header and a class-level description comment:

```boxlang
/**
 * Copyright Since 2005 ColdBox Framework by Luis Majano and Ortus Solutions, Corp
 * www.ortussolutions.com
 * ---
 * MCP Tools for ColdBox <Subsystem> introspection.
 */
```

## Skills

Skills live in `.agents/skills` and provide specialized instructions for common tasks. Load them via the `skill` tool when a task matches their domain.

### ColdBox Skills

- **coldbox-async-programming** — Async pipelines, Futures, `all()`/`allApply()`/`anyOf()`, executor management, and the `async()` helper.
- **coldbox-cli** — Application scaffolding, artifact generation (handlers, models, services, views, layouts, interceptors, modules, ORM, tests), language flags (BoxLang vs CFML), AI integration commands, and feature flags (--docker, --vite, --rest, --migrations, --ai).
- **coldbox-configuration** — `ColdBox.cfc` configuration: core framework settings, environments, modules, interceptors, LogBox, CacheBox, and WireBox config. Applies to `**/*.{bx,bxm,cfc,cfm,cfml}`.
- **coldbox-di** — ColdBox-specific injection DSL namespaces (`coldbox:`, `logbox:`, `cachebox:`, `wirebox:`, `model:`), module settings injection, injection of ColdBox services (interceptors, flash, router, scheduler), and the enhanced ColdBox binder helpers. Applies to `**/*.{bx,bxm,cfc,cfm,cfml}`.
- **coldbox-event-model** — Event object (`prc`/`rc`), request collections, view rendering, `renderData()`, `relocate()`, HTTP status codes, and event caching. Applies to `**/*.{bx,bxm,cfc,cfm,cfml}`.
- **coldbox-handler-development** — Handler (controller) creation, CRUD actions, dependency injection in handlers, `preHandler`/`postHandler` advices, secured actions, REST handlers, and the EventHandler base class. Applies to `**/*.{bx,bxm,cfc,cfm,cfml}`.
- **coldbox-interceptor-development** — Interceptor creation for cross-cutting concerns, lifecycle event listening, security checks, logging, CORS, rate limiting, request/response transformation, and custom interception points. Applies to `**/*.{bx,bxm,cfc,cfm,cfml}`.
- **coldbox-layout-development** — Layout (master page) templates, admin layouts, reusable view partials, nested content rendering with `renderView()`, dynamic layout switching. Applies to `**/*.{bx,bxm,cfc,cfm,cfml}`.
- **coldbox-logging** — LogBox configuration inside ColdBox (`config/LogBox.cfc`, inline DSL, configFile), per-environment log levels, injecting loggers via WireBox `logbox` DSL, accessing LogBox from the ColdBox proxy. Applies to `**/*.{bx,cfc,cfm,bxm}`.
- **coldbox-module-development** — Reusable ColdBox modules, `ModuleConfig.cfc`, module-specific routes/models/settings/interceptors, ForgeBox packaging, internal sub-system modules. Applies to `**/*.{bx,bxm,cfc,cfm,cfml}`.
- **coldbox-request-context** — `RequestContext` event object, `rc`/`prc` collection management, `buildLink()`, HTTP method detection, request metadata, flash scope, AJAX handling, HTTP headers and body. Applies to `**/*.{bx,bxm,cfc,cfm,cfml}`.
- **coldbox-rest-api-development** — RESTful API development with `RestHandler`, CRUD endpoints, API versioning, JWT/bearer authentication, structured error responses, resource representations with mementos. Applies to `**/*.{bx,bxm,cfc,cfm,cfml}`.
- **coldbox-routing-development** — URL routing configuration, RESTful resource routes, route groups, URL pattern matching with constraints, named routes, `Router.cfc`. Applies to `**/*.{bx,bxm,cfc,cfm,cfml}`.
- **coldbox-scheduled-tasks** — Task scheduler creation (`config/Scheduler.cfc`), task registration, frequency constraints (cron/duration), lifecycle callbacks (`before`/`after`/`onFailure`/`onSuccess`), server fixation for clustered apps, module schedulers. Applies to `**/*.{bx,bxm,cfc,cfm,cfml}`.
- **coldbox-testing-base-classes** — Testing base class selection: `BaseTestCase` (integration), `BaseHandlerTest` (isolated handler), `BaseModelTest` (model unit), `BaseInterceptorTest` (interceptor unit); bundle annotations (`appMapping`, `configMapping`, `unloadColdBox`, etc.); test harness setup.
- **coldbox-testing-handler** — Handler testing with `execute()`, `rc`/`prc` assertions, view selection verification, relocation mocking, `renderData()` testing, `getHandlerResults()`, HTTP method/header simulation, `BaseHandlerTest`. Applies to `**/*.{bx,bxm,cfc,cfm,cfml}`.
- **coldbox-testing-http-methods** — HTTP method simulation in tests: `get()`/`post()`/`put()`/`patch()`/`delete()`/`request()`, request headers (Authorization, Accept, Content-Type), JSON request body, `toHaveStatus()`, `toHaveInvalidData()`, vs `execute()` event-based testing. Applies to `**/*.{bx,bxm,cfc,cfm,cfml}`.
- **coldbox-testing-integration** — Integration testing with real dependencies (database, WireBox, ColdBox context), full request/response cycles with `execute()`, test database setup/teardown, service testing with real data, `BaseTestCase`. Applies to `**/*.{bx,bxm,cfc,cfm,cfml}`.
- **coldbox-testing-interceptor** — Interceptor unit testing with `BaseInterceptorTest`, pre-wired mocks (`mockController`, `mockRequestService`, `mockLogger`, `mockLogBox`, `mockFlash`), interceptor properties, interception point invocation. Applies to `**/*.{bx,bxm,cfc,cfm,cfml}`.
- **coldbox-testing-model** — Model/service unit testing with `BaseModelTest`, pre-wired mocks (`mockLogger`, `mockLogBox`, `mockCacheBox`, `mockWireBox`), `model.init()`, `prepareMock()`, CommandBox scaffolding. Applies to `**/*.{bx,bxm,cfc,cfm,cfml}`.
- **coldbox-view-rendering** — View rendering and partials, reusable view components, view output caching, passing data to views, `renderView()` from services, dynamic view selection. Applies to `**/*.{bx,bxm,cfc,cfm,cfml}`.

### BoxLang Language Skills

- **boxlang-application-descriptor** — `Application.bx` behavior: app discovery and nesting, multi-application isolation, lifecycle events, pseudo-constructor settings, session management, mappings/javaSettings resolution, app-scoped schedulers/watchers.
- **boxlang-async-programming** — `BoxFuture`, `futureNew`, `asyncRun`, `asyncAll`/`asyncAny`/`asyncAllApply`, executors, schedulers, thread components, parallel pipelines, file watchers, distributed locking with `bx:lock`.
- **boxlang-best-practices** — Naming conventions, structural patterns, scoping, error handling, performance optimization, maintainability patterns for idiomatic BoxLang code.
- **boxlang-caching** — Cache providers, `cachePut`/`cacheGet` BIFs, output caching, cache regions, distributed caching (Redis, Couchbase), TTL policies, distributed locking.
- **boxlang-classes-and-oop** — Class/component/interface definitions, inheritance hierarchies, annotations, properties, constructors, abstract classes, static members, object-oriented design patterns in BoxLang.
- **boxlang-configuration** — `boxlang.json` runtime settings, environment variable overrides, datasource/cache/executor/module/logging/security/scheduler/watcher configuration.
- **boxlang-database-access** — `queryExecute`, `bx:query`, datasource configuration, parameterized queries, transactions, stored procedures, query manipulation, SQL injection prevention.
- **boxlang-file-handling** — `fileRead`/`fileWrite`/`fileCopy`/`fileMove`, `directoryList`/`directoryCreate`, `fileUpload`, streaming large files, CSV/JSON processing from disk.
- **boxlang-file-watchers** — `watcherNew`/`watcherStart`/`watcherStop` lifecycle BIFs, `WatcherInstance` APIs, event payload handling, recursive directory monitoring, debounce/throttle/atomicWrites tuning, errorThreshold auto-stop, listener patterns.
- **boxlang-functional-programming** — Lambdas, closures, arrow functions, higher-order functions, functional array/struct pipelines (`map`, `filter`, `reduce`, `flatMap`, `groupBy`), destructuring, spread syntax, Java Streams integration.
- **boxlang-interceptors** — Creating interceptors, registering announcement points, `announce()`/`announceAsync()`, pre/post operation hooks, validation interceptors, security guards, `BoxRegisterInterceptor()` BIF.
- **boxlang-java-integration** — `createObject`, static method calls, type conversion, importing classes, closures as functional interfaces, JAR inclusion, JSR-223 scripting, Java libraries from BoxLang.
- **boxlang-language-fundamentals** — BoxLang syntax, file types (`.bx`/`.bxs`/`.bxm`), variables, scopes, operators, control flow, exception handling, type system, destructuring, spread syntax.
- **boxlang-modules-and-packages** — Module installation (`box install`), `boxlang.json` module settings, BoxLang+ premium modules (bx-pdf, bx-redis, bx-csv, bx-spreadsheet), CFML compatibility, ORM, mail, module introspection.
- **boxlang-runtime-cli-scripting** — BoxLang CLI scripts and classes, command-line arguments, REPL, action commands (compile, cftranspile, featureaudit), runtime context detection, CLI-specific BIFs.
- **boxlang-runtime-miniserver** — BoxLang MiniServer (Undertow-based), `miniserver.json`, `.boxlang.json` project convention, health checks, warmup URLs, environment variable overrides.
- **boxlang-scheduled-tasks** — Scheduler DSL classes, `BaseScheduler`/`ScheduledTask` fluent APIs, `schedulerStart`/`schedulerGet`/`schedulerStats` BIFs, cron expressions, frequency constraints, lifecycle callbacks, `bx:schedule` component.
- **boxlang-security** — Security vulnerability review, runtime security configuration, injection prevention (SQL, XSS, CSRF), file upload safety, secrets management, authentication patterns, session security, OWASP Top 10.
- **boxlang-templating** — `.bxm` template files, HTML with BoxLang output expressions, template components (`bx:output`, `bx:loop`, `bx:if`, `bx:include`, `bx:script`), views.
- **boxlang-web-development** — `Application.bx` lifecycle, request/response handling, sessions, forms, REST APIs, HTTP clients, routing, CSRF protection, Server-Sent Events, CommandBox/MiniServer configuration.
- **boxlang-zip** — `bx:zip` component for creating, extracting, listing, and modifying ZIP archives: compression, filtering, encryption, backup/restore workflows.

### TestBox Testing Skills

- **testbox-assertions** — `$assert` object for xUnit-style assertions: `isTrue`, `isEqual`, `includes`, `isEmpty`, `key`, `instanceOf`, `throws`, `between`, `closeTo`, `lengthOf`, `match`, `null`, `typeOf`; custom assertion registration with `addAssertions()`. Applies to `**/tests/**/*.{bx,bxm,cfc,cfm,cfml}`.
- **testbox-bdd** — BDD-style testing with `describe`/`it` blocks, Gherkin-style (`feature`/`story`/`scenario`/`given`/`when`/`then`), lifecycle hooks (`beforeAll`/`afterAll`/`beforeEach`/`afterEach`/`aroundEach`), focused/skipped specs, data binding, `asyncAll` parallel specs. Applies to `**/tests/**/*.{bx,bxm,cfc,cfm,cfml}`.
- **testbox-cbmockdata** — Realistic fake data generation: age, boolean, date, datetime, email, name, sentence, SSN, UUID, URL, lorem ipsum, bacon lorem, image URL, IP address, autoincrement, `oneof`, `rnd`/`rand`; arrays of objects, nested objects, custom supplier closures. Applies to `**/tests/**/*.{bx,bxm,cfc,cfm,cfml}`.
- **testbox-expectations** — Fluent expectations with `expect()`/`expectAll()`, all built-in matchers (`toBe`, `toBeTrue`, `toBeArray`, `toHaveKey`, `toThrow`, `toMatch`, `toBeBetween`, `toInclude`, etc.), `not` operator (`notToBe`), matcher chaining, custom matchers with `addMatchers()`. Applies to `**/tests/**/*.{bx,bxm,cfc,cfm,cfml}`.
- **testbox-listeners** — Run listener callbacks: `onBundleStart`/`onBundleEnd`, `onSuiteStart`/`onSuiteEnd`, `onSpecStart`/`onSpecEnd`; custom progress indicators, loggers, live dashboards; passing listeners to `run()`/`runRaw()`. Applies to `**/tests/**/*.{bx,bxm,cfc,cfm,cfml}`.
- **testbox-mockbox** — Mocks, stubs, spies: `createMock()`/`createEmptyMock()`/`prepareMock()`, `$()` method stubbing, `$args()`/`$results()`/`$throws()`, call count verification (`$once()`/`$never()`/`$times()`/`$atLeast()`/`$atMost()`), `$callLog()`, `$property()`, `querySim()`, `$spy()`. Applies to `**/tests/**/*.{bx,bxm,cfc,cfm,cfml}`.
- **testbox-reporters** — Reporter selection: ANTJunit, Console, Doc, JSON, JUnit, Min, MinText, Simple, Text, XML, Streaming; reporter options (`hideSkipped`, editor links); custom reporters via `IReporter`. Applies to `**/tests/**/*.{bx,bxm,cfc,cfm,cfml}`.
- **testbox-runners** — Test execution: CommandBox CLI (`testbox run`), BoxLang CLI (`./testbox/run`), HTML web runner, programmatic (`run`/`runRaw`/`runRemote`), streaming runner, watcher mode, CLI flags (`--show-failed-only`, `--dry-run`, `--slow-threshold-ms`, `--stacktrace`, `--max-failures`). Applies to `**/tests/**/*.{bx,bxm,cfc,cfm,cfml}`.
- **testbox-unit-xunit** — xUnit-style tests: `testXxx()` methods, `setup()`/`teardown()` lifecycle, `beforeTests()`/`afterTests()`, `$assert` object, Arrange-Act-Assert (AAA) pattern. Applies to `**/tests/**/*.{bx,bxm,cfc,cfm,cfml}`.
- **testing-coverage** — Code coverage analysis setup, coverage reporting configuration, CI pipeline integration, TestBox coverage options, coverage metrics interpretation, improving coverage of untested code paths. Applies to `**/tests/**/*.{bx,bxm,cfc,cfm,cfml}`.
- **testing-fixtures** — Test fixtures, factory patterns, test data builders, shared fixture files, factory overrides, `cbMockData` integration, test data setup/teardown. Applies to `**/tests/**/*.{bx,bxm,cfc,cfm,cfml}`.

### WireBox Skills

- **wirebox-aop** — AOP engine activation (Mixer listener), method interceptor aspects, `mapAspect()`/`bindAspect()`, Matcher DSL (`any`, `regex`, `mappings`, `instanceOf`, `annotatedWith`, `methods`, `returns`), auto-aspect binding via annotations, `MethodInvocation` API (`proceed`, `getArgs`, `getTarget`, `getMethod`), built-in aspects (CFTransaction, HibernateTransaction, MethodLogger). Applies to `**/*.{bx,bxm,cfc,cfm,cfml}`.

### General / Cross-Cutting Skills

- **code-documenter** — Developer-facing documentation: inline comments, docstrings, API references, onboarding guides, runbooks, consistency audits.
- **code-reviewer** — Code review for correctness, security, maintainability, performance, test coverage risk; PR review, architecture drift detection, bug risk assessment, severity-ranked findings.
- **github-action-authoring** — Composite GitHub Actions: platform support (Windows/Linux/macOS), PATH issues, action inputs/outputs/steps, PowerShell steps, installer flag ordering, CI test jobs.
- **java-expert** — Java service/library/backend implementation, API design, concurrency, performance profiling, dependency management, testing strategy, production hardening.
- **junit-expert** — JUnit 5 tests: lifecycle, parameterized tests, extensions, assertions, parallel execution, dynamic tests, build tool integration.
- **ortus-coding-standards** — Ortus Solutions coding standards for BoxLang, CFML, and Java: indentation (tabs), spacing, brace placement, naming, alignment, comments, and structural conventions.
- **security-expert** — Secure system design, threat modeling, defense-in-depth, authentication, authorization, secrets handling, input validation, secure coding standards, remediation planning.

## Custom Skills

This project does not currently define custom project-specific skills. The `.agents/skills/` directory is populated entirely from the skills-lock.json manifest, which pulls from the `ortus-boxlang/skills` GitHub repository.

When adding custom skills, place them in `.agents/skills-custom/` alongside existing skill directories, and update this section to document their purpose and trigger conditions.