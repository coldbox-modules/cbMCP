<p align="center">
	<img src="https://www.ortussolutions.com/__media/coldbox-185-logo.png">
	<br>
	<img src="https://www.ortussolutions.com/__media/wirebox-185.png" height="125">
	<img src="https://www.ortussolutions.com/__media/cachebox-185.png" height="125" >
	<img src="https://www.ortussolutions.com/__media/logbox-185.png"  height="125">
</p>

<p align="center">
	Copyright Since 2005 ColdBox Platform by Luis Majano and Ortus Solutions, Corp
	<br>
	<a href="https://www.coldbox.org">www.coldbox.org</a> |
	<a href="https://www.ortussolutions.com">www.ortussolutions.com</a>
</p>

----

# cbMCP — ColdBox MCP Server

`cbMCP` is a ColdBox module that exposes your running ColdBox application as an **MCP (Model Context Protocol) server**. It gives AI clients (Claude Desktop, VS Code Copilot, Cursor, etc.) live, read-only introspection tools for the entire ColdBox ecosystem — routing, handlers, modules, WireBox, CacheBox, LogBox, schedulers, async executors, and more.

## Requirements

- **BoxLang 1.x** (recommended) or ColdBox 8+
- **BoxLang AI Module** (`bx-ai`)
- `box` (CommandBox) for installation

## Installation

```bash
box install bx-ai
box install cbmcp
```

## Configuration

All settings are configured in the module's configuration struct inside your application's `ColdBox.cfc` under `modules.cbMCP.settings`.

```js
// config/ColdBox.cfc
moduleSettings = {
    cbMCP = {
        settings = {
            authToken          : [],
            securityProfiles   : {
                admin    : { includedTools: [ "*" ],                                              excludedTools: [] },
                readonly : { includedTools: [ "*_get*", "*_has*", "*_search*", "*_read*" ],       excludedTools: [] }
            },
            allowedIPs         : [ "127.0.0.1" ],
            corsAllowedOrigins : [],
            enableStats        : true,
            maxRequestBodySize : 0,
            includedTools      : [ "*" ],
            excludedTools      : []
        }
    }
}
```

No extra configuration is required beyond this. Once the module is installed and your application boots, the MCP endpoint is live at:

```
http://<host>:<port>/cbmcp
```

The server auto-scans all tool classes under `cbMCP.models.tools` and registers every `@mcpTool`-annotated function.

### Settings Summary

| Setting | Type | Default | Description |
|---|---|---|---|
| `authToken` | string \| array | `[]` | Bearer token(s) controlling access. Supports a simple string or an array of structs with per-token tool filters. Empty = no auth. See [Authentication & Access Control](#authentication--access-control). |
| `securityProfiles` | struct | `{ admin: { includedTools: ["*"], excludedTools: [] }, readonly: { includedTools: ["*_get*","*_has*","*_search*","*_read*"], excludedTools: [] } }` | Named security profiles that define reusable tool access policies. Profiles are referenced from `authToken` entries via the `profile` field. Two built-in profiles (`admin` and `readonly`) are always available and can be overridden. See [Security Profiles](#security-profiles). |
| `allowedIPs` | array | `["127.0.0.1"]` | IP allowlist. Supports individual IPs and CIDR ranges (`192.168.0.0/24`). Empty array = all IPs allowed. |
| `corsAllowedOrigins` | array | `[]` | CORS allowed origins. Supports wildcards (`*.domain.com`). Empty = no CORS headers. |
| `enableStats` | boolean | `true` | Enable MCP server statistics tracking (tool call counts, timing). |
| `maxRequestBodySize` | numeric | `0` | Max HTTP request body size in bytes. `0` = no limit. |
| `includedTools` | array | `["*"]` | Tool whitelist. `["*"]` = all tools. Supports exact names and glob patterns (`handler*`, `cachebox_get*`). |
| `excludedTools` | array | `[]` | Tools to hide from the MCP client after the whitelist is applied. Supports exact names and glob patterns. |

### Security Notes

- **`authToken`**: Strongly recommended for any non-localhost deployment. Clients must send `Authorization: Bearer {token}`.
- **`allowedIPs`**: Defaults to `localhost` only. For access from Docker containers or remote machines, add their IPs or CIDR ranges.
- **`excludedTools`**: Use this to hide sensitive operations (e.g., `system_reinit_app`, `cachebox_clear_all`) from MCP clients. Glob patterns like `cachebox_clear*` are supported.

---

## Authentication & Access Control

The `authToken` setting supports two shapes.

### Shape 1 — Simple string

One token, full access to every registered tool:

```js
moduleSettings = {
    cbMCP = {
        settings = {
            authToken : "my-secret-token"
        }
    }
}
```

Clients send:

```
Authorization: Bearer my-secret-token
```

### Shape 2 — Array of structs

Multiple tokens, each with independent tool-level access control:

```js
moduleSettings = {
    cbMCP = {
        settings = {
            authToken : [
                { token: "admin-token",     includedTools: ["*"],                              excludedTools: [] },
                { token: "readonly-token",  includedTools: ["*_get*", "*_has*"],              excludedTools: [] },
                { token: "ops-token",       includedTools: ["*"],                              excludedTools: ["system_reinit_app", "cachebox_clear_all"] }
            ]
        }
    }
}
```

Each struct requires a `token` field. Both `includedTools` and `excludedTools` are optional:

| Field | Default | Description |
|---|---|---|
| `token` | *(required)* | The Bearer token value the client must send. |
| `profile` | `"admin"` | Named security profile to apply. References a profile defined in `securityProfiles`. See [Security Profiles](#security-profiles). |
| `includedTools` | `["*"]` | Tool whitelist. `["*"]` means all tools. Provide specific names to restrict access. Overrides the profile's `includedTools` when set. |
| `excludedTools` | `[]` | Tools to block even if they match the `includedTools` whitelist. Merged with the profile's `excludedTools`. |

**Filtering rules (applied in order):**

1. If `includedTools` does **not** contain `"*"` and the tool name does not match any pattern → **denied**.
2. If the tool name matches any pattern in `excludedTools` → **denied**.
3. Otherwise → **allowed**.

Both `includedTools` and `excludedTools` support **glob patterns**:

| Pattern | Matches |
|---|---|
| `"*"` | All tools |
| `"handler*"` | All tools starting with `handler_` |
| `"cachebox_get*"` | `cachebox_get_all`, `cachebox_get_stats`, `cachebox_get_keys`, … |
| `"*_health*"` | Any tool with `_health` in its name |
| `"system_get_setting"` | Exact match only |

### Security Profiles

Security profiles let you define reusable named access policies that can be referenced from multiple `authToken` entries via the `profile` field. This avoids repeating the same `includedTools`/`excludedTools` patterns across tokens.

#### Built-in Profiles

Two profiles are built in and always available (you can override them in `securityProfiles`):

| Profile | includedTools | excludedTools | Use Case |
|---|---|---|---|
| `admin` | `["*"]` | `[]` | Unrestricted access to every tool |
| `readonly` | `["*_get*", "*_has*", "*_search*", "*_read*"]` | `[]` | Read-only observability — all inspection tools, no mutation |

#### Custom Profiles

Define additional profiles in the `securityProfiles` setting:

```js
moduleSettings = {
    cbMCP = {
        settings = {
            securityProfiles : {
                operator : {
                    includedTools : ["*_get*", "*_has*", "system_reinit_app", "scheduler_*"],
                    excludedTools : []
                }
            }
        }
    }
}
```

#### Referencing a Profile from a Token

Each `authToken` entry can reference a profile using the `profile` field:

```js
moduleSettings = {
    cbMCP = {
        settings = {
            authToken : [
                { token: "admin-token",    profile: "admin" },
                { token: "readonly-token", profile: "readonly" },
                { token: "ops-token",      profile: "operator", excludedTools: ["system_reinit_app"] }
            ]
        }
    }
}
```

When a `profile` is specified:

1. The profile's `includedTools` and `excludedTools` are loaded as the baseline.
2. If the token entry also has its own `includedTools`, it **overrides** the profile's `includedTools`.
3. If the token entry has its own `excludedTools`, they are **merged** with the profile's `excludedTools`.
4. The standard filtering rules (whitelist → blacklist) are then applied to the merged set.

#### Profile Inheritance Rules

| Token has `profile` | Token has `includedTools` | Token has `excludedTools` | Result |
|---|---|---|---|
| Yes | No | No | Profile's settings used as-is |
| Yes | Yes | No | Token's `includedTools` overrides profile |
| Yes | No | Yes | Profile's `includedTools` + merged `excludedTools` |
| Yes | Yes | Yes | Token's `includedTools` overrides profile + merged `excludedTools` |
| No | Yes/No | Yes/No | No profile applied; token's own settings used directly |

### Disabling authentication

Leave `authToken` empty (`""` or `[]`) or omit it entirely to run in open-access mode (no `Authorization` header required). This is suitable only for localhost-only deployments already protected by `allowedIPs`.

---

## Connecting an AI Client

### Claude Desktop

Add the following entry to your `claude_desktop_config.json` (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):

```json
{
  "mcpServers": {
    "cbMCP": {
      "command": "npx",
      "args": ["-y", "@depasquale/mcp-http-stdio-bridge", "--url", "http://127.0.0.1:<port>/cbmcp"]
    }
  }
}
```

Replace `<port>` with the port your ColdBox application is running on (e.g. `60299` during local development). The bridge package (`@depasquale/mcp-http-stdio-bridge`) is installed automatically via `npx` on first use — no manual install needed.

**Example (local BoxLang dev server on port 60299):**

```json
{
  "mcpServers": {
    "cbMCP": {
      "command": "npx",
      "args": ["-y", "@depasquale/mcp-http-stdio-bridge", "--url", "http://127.0.0.1:60299/cbmcp"]
    }
  }
}
```

After saving, restart Claude Desktop. The `cbMCP` server will appear in the tools panel.

### VS Code / GitHub Copilot

Add to your `.vscode/mcp.json` (or user-level MCP settings):

```json
{
  "servers": {
    "cbMCP": {
      "type": "http",
      "url": "http://127.0.0.1:<port>/cbmcp"
    }
  }
}
```

### Any MCP-compatible client (HTTP/SSE transport)

Point the client directly at the SSE endpoint:

```
http://<host>:<port>/cbmcp
```

---

## MCP Tools (42 Total)

The server exposes tools across 11 ColdBox domains. Each tool is an `@mcpTool`-annotated method discovered automatically via classpath scanning.

### System & Runtime — `SystemTools`

| Tool | Description |
|---|---|
| `system_get_time` | Server's current date/time in ISO 8601 format |
| `system_get_coldbox_settings` | All ColdBox framework settings and configuration |
| `system_get_runtime_info` | Engine name (boxlang/lucee/adobe), version, app layout (flat/boxlang), OS, Java info |
| `system_get_application_structure` | Application directory layout (handlers, models, views, tests, modules_app) — adapts to both flat and boxlang layouts |
| `system_get_setting( name )` | Returns a single ColdBox application setting by key |
| `system_reinit_app` | Reinitialises the ColdBox application (fires preReinit interceptors, shuts down services, removes controller) |

### Handlers — `HandlerTools`

| Tool | Description |
|---|---|
| `handler_get_all` | All registered event handlers with their public actions — returns core handlers and per-module handlers |
| `handler_get( handlerName, [moduleName] )` | Detailed metadata for a specific handler: invocation path, public actions, parent class |

### Routing — `RoutingTools`

| Tool | Description |
|---|---|
| `routing_get_all` | All registered application routes with patterns and targets |
| `routing_get_module_names` | Lists which modules have routes registered in the module routing table |
| `routing_get_module( moduleName )` | Routes registered by a specific module |
| `routing_get_settings` | Router configuration: base URL, loose matching, multi-domain routing, route counts |

### Modules — `ModuleTools`

| Tool | Description |
|---|---|
| `module_get_all` | Full metadata for all registered modules: activation status, versions, paths, settings, conventions, dependencies, entry points, model mappings |
| `module_get_names` | Flat list of activated module names (sorted) |
| `module_get( moduleName )` | Full configuration and status for a specific registered module |

### WireBox — `WireBoxTools`

| Tool | Description |
|---|---|
| `wirebox_get_mappings` | All registered DI mappings: path, type, scope, autowire, alias, delegates, thread safety, virtual inheritance |
| `wirebox_has_mapping( name )` | Checks whether a specific DI mapping exists |
| `wirebox_get_mapping( name )` | Full mapping definition for a specific binding |

### CacheBox — `CacheBoxTools`

| Tool | Description |
|---|---|
| `cachebox_get_all` | All registered cache providers with stats (hits, misses, evictions, performance ratio) and configuration |
| `cachebox_get_names` | Sorted list of cache provider names |
| `cachebox_get_stats( cacheName )` | Detailed hit/miss/eviction/performance statistics for a cache |
| `cachebox_get_keys( cacheName )` | All keys currently stored in a cache |
| `cachebox_get_size( cacheName )` | Number of objects in a cache |
| `cachebox_get_key_metadata( cacheName, objectKey )` | Metadata for a single cached entry |
| `cachebox_has_key( cacheName, objectKey )` | Existence check for a specific key |
| `cachebox_get_store_metadata_report( cacheName )` | Full store metadata for all entries in a cache |
| `cachebox_clear_all( cacheName )` | Clear all entries from a cache |
| `cachebox_clear_item( cacheName, objectKey )` | Remove a single entry by key |
| `cachebox_clear_all_events( cacheName, [async] )` | Clear all event-cached entries (IColdBoxProvider only) |
| `cachebox_clear_event( cacheName, eventSnippet, [queryString] )` | Clear a specific event cache entry |
| `cachebox_clear_event_multi( cacheName, eventSnippets, [queryString] )` | Clear multiple event cache entries at once |
| `cachebox_clear_all_views( cacheName, [async] )` | Clear all view-cached entries (IColdBoxProvider only) |
| `cachebox_clear_view( cacheName, viewSnippet )` | Clear a specific view cache entry |
| `cachebox_clear_view_multi( cacheName, viewSnippets )` | Clear multiple view cache entries at once |

### LogBox — `LogBoxTools`

| Tool | Description |
|---|---|
| `logbox_get_info` | LogBox version, ID, registered logger names, and appender names |
| `logbox_get_loggers` | All registered loggers with category, min/max levels, and appender names |
| `logbox_get_logger( category )` | Details for a specific logger category |
| `logbox_get_appenders` | All registered appenders: name, class, min/max levels, initialization status |
| `logbox_get_root_logger` | Root logger details and attached appenders |
| `logbox_read_entries( [appenderName], [limit=100] )` | Last N log entries from file-based appenders (FileAppender / RollingFileAppender) |
| `logbox_get_last_error( [appenderName] )` | Most recent ERROR or FATAL entry from log files |

### Interceptors — `InterceptorTools`

| Tool | Description |
|---|---|
| `interceptor_get_all` | All registered interception points with their interceptor pools (name, path, count) |
| `interceptor_get_states` | Array of all registered interception state names only |

### Schedulers — `SchedulerTools`

| Tool | Description |
|---|---|
| `scheduler_get_all` | All registered application schedulers with tasks, stats, started status, server fixation, timezone |
| `scheduler_get( schedulerName )` | Details for a specific scheduler: tasks, task stats, start time, executor, timezone |
| `scheduler_get_task_stats( schedulerName, taskName )` | Execution statistics for a specific task |
| `scheduler_run_task( schedulerName, taskName )` | Execute a scheduled task on demand (force-run, bypasses constraints) |
| `scheduler_pause_task( schedulerName, taskName )` | Pause a scheduled task so it skips execution until resumed |
| `scheduler_resume_task( schedulerName, taskName )` | Resume a previously paused scheduled task |

### Async Executors — `AsyncTools`

| Tool | Description |
|---|---|
| `async_get_all` | All registered async executors with status, thread pool info, and task statistics |
| `async_get_names` | Sorted list of registered executor names |

---

## MCP Resources

`cbMCP` also exposes **MCP Resources** — ambient read-only context automatically injected into AI conversations:

| URI | Description |
|---|---|
| `coldbox://app/settings` | All active ColdBox application configuration settings (key-value, filtered to serializable values) |
| `coldbox://app/modules` | All currently activated HMVC modules with name, version, description, entry point, author, status |
| `coldbox://app/routes` | All registered URL routes with pattern, handler, action, name, HTTP methods, module, namespace |
| `coldbox://app/handlers` | Sorted list of all registered event handler names in the application |

---

## MCP Prompts (5 Total)

The server registers pre-built MCP prompts that guide AI clients through common ColdBox diagnostic and administrative workflows.

| Prompt | Description | Arguments |
|---|---|---|
| `coldbox_app_overview` | Comprehensive overview of the ColdBox application: architecture, modules, routing strategy, notable configuration | — |
| `debug_handler` | Diagnose issues with a specific ColdBox event handler: routing, dependency injection, event execution lifecycle | `handlerName` *(required)* |
| `cache_health_report` | CacheBox health report: hit rates, eviction counts, object counts, identify degraded or pressured providers | — |
| `interceptor_audit` | Full interceptor audit: registered interception points, listener pools, ordering improvements, conflict detection | — |

---

## Client Interaction Examples

All interactions use JSON-RPC 2.0 over HTTP POST to `/cbmcp`.

> Replace `http://localhost:{port}` with your server URL. Add `Authorization: Bearer {token}` if `authToken` is configured.

### 1. List all available tools

```bash
curl -s http://localhost:60299/cbmcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/list","id":"1"}' | jq '.result.tools[] | {name, description}'
```

### 2. Get ColdBox runtime info

```bash
curl -s http://localhost:60299/cbmcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"system_get_runtime_info","arguments":{}},"id":"2"}' | jq '.result.content[0].text | fromjson'
```

### 3. List all registered handlers

```bash
curl -s http://localhost:60299/cbmcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"handler_get_all","arguments":{}},"id":"3"}' | jq '.result.content[0].text | fromjson'
```

### 4. Get all application routes

```bash
curl -s http://localhost:60299/cbmcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"routing_get_all","arguments":{}},"id":"4"}' | jq '.result.content[0].text | fromjson'
```

### 5. Check a WireBox mapping

```bash
curl -s http://localhost:60299/cbmcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"wirebox_has_mapping","arguments":{"name":"MyService"}},"id":"5"}' | jq '.result.content[0].text | fromjson'
```

### 6. Get cache statistics

```bash
curl -s http://localhost:60299/cbmcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"cachebox_get_stats","arguments":{"cacheName":"default"}},"id":"6"}' | jq '.result.content[0].text | fromjson'
```

### 7. Get scheduler info and run a task

```bash
curl -s http://localhost:60299/cbmcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"scheduler_run_task","arguments":{"schedulerName":"myScheduler","taskName":"dailyReport"}},"id":"7"}' | jq '.result.content[0].text | fromjson'
```

### 8. Read recent log entries

```bash
curl -s http://localhost:60299/cbmcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"logbox_read_entries","arguments":{"limit":20}},"id":"8"}' | jq '.result.content[0].text | fromjson'
```

> **Tip:** Pipe through `jq` to extract and parse the JSON response. The MCP server wraps tool results inside `result.content[0].text` as a JSON-encoded string.

### Calling a Prompt

```bash
curl -s http://localhost:60299/cbmcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"prompts/get","params":{"name":"coldbox_app_overview","arguments":{}},"id":"9"}' | jq '.result.messages'
```

---

## Learning ColdBox

ColdBox is the defacto standard for building modern ColdFusion (CFML) applications. It has the most extensive [documentation](https://coldbox.ortusbooks.com) of all modern web application frameworks.

If you don't like reading so much, then you can try our video learning platform: [CFCasts (www.cfcasts.com)](https://www.cfcasts.com)

## Ortus Sponsors

ColdBox is a professional open-source project and it is completely funded by the [community](https://patreon.com/ortussolutions) and [Ortus Solutions, Corp](https://www.ortussolutions.com). Ortus Patreons get many benefits like a cfcasts account, a FORGEBOX Pro account and so much more. If you are interested in becoming a sponsor, please visit our patronage page: [https://patreon.com/ortussolutions](https://patreon.com/ortussolutions)

### THE DAILY BREAD

> "I am the way, and the truth, and the life; no one comes to the Father, but by me (JESUS)" Jn 14:1-12
