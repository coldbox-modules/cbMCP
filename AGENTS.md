# Agents

**cbMCP** is a ColdBox module that implements a [Model Context Protocol (MCP)](https://modelcontextprotocol.io) server, exposing ColdBox application internals — settings, modules, routes, handlers, caches, interceptors, WireBox bindings, log appenders, and scheduled tasks — as structured MCP **tools**, **resources**, and **prompts**. This allows AI clients (Claude, Copilot, etc.) to introspect and interact with a running ColdBox application in real time.

---

## Agents Registry

Agents are high-level orchestration components that leverage specialized **Skills** to perform complex tasks autonomously.

| Agent Name | Description |
| :--- | :--- |
| `Explore` | Fast read-only codebase exploration and Q&A. Use for context gathering before implementation. |
| `modernize` | Orchestrates assess → plan → execute workflow for upgrading or migrating applications. |
| `modernize-azure-java` | Migrates Java applications to Azure. |
| `modernize-java` | Upgrades Java/Spring Boot projects to target versions via incremental planning. |
| `modernize-java-upgrade` | Dedicated version-specific upgrade specialist for Java and Spring Boot. |
| `modernize-java-security` | Scans and remediates CVE vulnerabilities in Java project dependencies. |
| `assessment-coordinator` | Coordinates the codebase assessment phase using MCP tools. |
| `planning-coordinator` | Generates `plan.md` and `tasks.json` from assessment results or direct task specs. |
| `execution-coordinator` | Coordinates the multi-agent execution phase. |
| `modernize-rearchitecture` | Leads multi-agent teams to modernize and rearchitect legacy applications. |
| `modernize-rearchitecture-worker` | Implements specific tasks within a rearchitecture workflow. |

---

## Agent Skills Registry

Skills are the granular domain-specific instruction sets used by agents to solve specific problems.

### BoxLang Skills

These skills provide deep expertise in the BoxLang language and its ecosystem.

| Skill Name | Description |
| :--- | :--- |
| `boxlang-application-descriptor` | Designing/debugging `Application.bx` behavior, lifecycle, and isolation. |
| `boxlang-async-programming` | Writing BoxLang asynchronous code: `BoxFuture`, executors, schedulers. |
| `boxlang-best-practices` | Improving BoxLang code for naming, structure, and maintainability. |
| `boxlang-caching` | Implementing BoxLang caching (providers, TTL, distributed caching). |
| `boxlang-classes-and-oop` | BoxLang classes, interfaces, inheritance, and design patterns. |
| `boxlang-configuration` | Configuring BoxLang runtime settings and datasources. |
| `boxlang-database-access` | Writing BoxLang database code and preventing SQL injection. |
| `boxlang-file-handling` | File and directory manipulation in BoxLang. |
| `boxlang-file-watchers` | Implementing BoxLang filesystem watchers. |
| `boxlang-functional-programming` | Lambdas, closures, and functional pipelines in BoxLang. |
| `boxlang-interceptors` | BoxLang's interceptor and event system. |
| `boxlang-java-integration` | Integrating BoxLang with Java (types, JSR-223, libraries). |
| `boxlang-language-fundamentals` | Syntax, variables, scopes, and control flow in BoxLang. |
| `boxlang-modules-and-packages` | Installing and configuring BoxLang modules. |
| `boxlang-runtime-cli-scripting` | CLI scripting and command-line arguments. |
| `boxlang-runtime-miniserver` | Running BoxLang as a lightweight web server. |
| `boxlang-scheduled-tasks` | Managing BoxLang scheduled workloads/tasks. |
| `boxlang-security` | Reviewing and implementing secure BoxLang patterns. |
| `boxlang-templating` | BoxLang markup templates and view building. |
| `boxlang-testing` | Testing BoxLang applications with TestBox. |
| `boxlang-web-development` | Web apps: routing, sessions, forms, and REST. |
| `boxlang-zip` | ZIP archive manipulation via `bx:zip`. |

### ColdBox Skills

These skills focus on the ColdBox framework features and its lifecycle.

| Skill Name | Description |
| :--- | :--- |
| `coldbox-async-programming` | ColdBox Futures and async pipelines. |
| `coldbox-cli` | Using CommandBox for ColdBox scaffolding and generation. |
| `coldbox-configuration` | Configuring ColdBox (LogBox, CacheBox, WireBox, etc.). |
| `coldbox-di` | Dependency Injection within ColdBox via WireBox. |
| `coldbox-event-model` | Request context (event), view rendering, and redirects. |
| `coldbox-handler-development` | Creating ColdBox handlers and CRUD actions. |
| `coldbox-interceptor-development` | Creating ColdBox interceptors for cross-cutting concerns. |
| `coldbox-layout-development` | Creating layouts, partials, and nested views. |
| `coldbox-logging` | Configuring and using LogBox for logging. |
| `coldbox-module-development` | Building reusable ColdBox modules. |
| `coldbox-request-context` | Managing `rc` and `prc` collections. |
| `coldbox-rest-api-development` | Building RESTful APIs and handling JWT. |
| `coldbox-routing-development` | Configuring ColdBox routes and pattern matching. |
| `coldbox-scheduled-tasks` | Managing ColdBox scheduled tasks. |
| `coldbox-testing-base-classes` | Choosing appropriate TestBox base classes. |
| `coldbox-testing-handler` | Testing ColdBox handlers in isolation. |
| `coldbox-testing-http-methods` | Simulating HTTP requests in tests. |
| `coldbox-testing-integration` | End-to-end integration testing. |
| `coldbox-testing-interceptor` | Unit testing ColdBox interceptors. |
| `coldbox-testing-model` | Unit testing ColdBox models/services. |
| `coldbox-view-rendering` | Rendering views and partials. |

### Specialized & Third-Party Skills

Domain-specific expertise for security, testing, and modernization.

| Skill Name | Description |
| :--- | :--- |
| `code-documenter` | Improving developer-facing documentation. |
| `code-reviewer` | Performing high-signal code reviews. |
| `github-action-authoring` | Authoring and debugging GitHub Actions. |
| `java-expert` | Java services, libraries, and JVM best practices. |
| `junit-expert` | Writing and structuring JUnit 5 tests. |
| `ortus-coding-standards` | Enforcing official Ortus Solutions coding standards. |
| `security-expert` | Secure software system design and review. |
| `testbox-assertions` | Using TestBox `$assert` for xUnit tests. |
| `testbox-bdd` | Writing BDD-style tests via TestBox. |
| `testbox-cbmockdata` | Generating fake/mock data via `cbMockData`. |
| `testbox-expectations` | Using fluent expectations in TestBox. |
| `testbox-listeners` | Implementing TestBox run listeners. |
| `testbox-mockbox` | Mocking/stubbing with MockBox. |
| `testbox-reporters` | Configuring TestBox reporters. |
| `testbox-runners` | Running TestBox test suites via CLI. |
| `testbox-unit-xunit` | Writing xUnit-style tests in TestBox. |
| `testing-coverage` | Setting up and interpreting code coverage. |
| `testing-fixtures` | Creating test fixtures and data builders. |
| `wirebox-aop` | Using WireBox Aspect-Oriented Programming. |
