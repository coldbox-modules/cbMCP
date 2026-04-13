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

- **BoxLang** 1.x (recommended) or ColdBox 8+
- **BoxLang AI Module**
- `box` (CommandBox) for installation

## Installation

```bash
box install bx-ai
box install cbmcp
```

## Configuration

No extra configuration is required. Once the module is installed and your application boots, the MCP endpoint is live at:

```
http://<host>:<port>/cbmcp
```

The server auto-scans all tool classes under `cbMCP.models.tools` and registers every `@mcpTool`-annotated function.

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

## Available Tools

### System (`SystemTools`)

| Tool | Description |
|------|-------------|
| `time_now` | Returns the server's current date/time (ISO 8601) |
| `get_coldbox_settings` | Returns all ColdBox framework settings |
| `get_runtime_info` | Engine name, version, app layout, and OS |
| `get_application_structure` | Application directory layout (handlers, models, views, etc.) |
| `get_setting( name )` | Returns a single ColdBox application setting by key |
| `reinit_app` | Reinitialises the ColdBox application |

### Handlers (`HandlerTools`)

| Tool | Description |
|------|-------------|
| `get_handlers` | Lists all registered event handlers with their actions |
| `get_handler( handlerName, [moduleName] )` | Returns metadata for a specific handler |

### Routing (`RoutingTools`)

| Tool | Description |
|------|-------------|
| `list_routes` | Lists all registered application routes |
| `list_module_routes` | Lists which modules have routes registered |
| `get_module_routes( moduleName )` | Returns routes registered by a specific module |
| `get_router_settings` | Returns router configuration (default route, SSL, etc.) |

### Modules (`ModuleTools`)

| Tool | Description |
|------|-------------|
| `get_modules` | Returns full metadata for all registered modules |
| `get_loaded_modules` | Returns a flat list of activated module names |
| `get_module( moduleName )` | Returns metadata for a specific module |

### WireBox (`WireBoxTools`)

| Tool | Description |
|------|-------------|
| `get_wirebox_mappings` | Returns all registered DI mappings |
| `has_mapping( name )` | Checks whether a mapping exists |
| `get_mapping( name )` | Returns the full mapping definition for a binding |

### CacheBox (`CacheBoxTools`)

| Tool | Description |
|------|-------------|
| `get_caches` | Returns all registered cache providers with stats |
| `get_cache_names` | Returns a flat list of cache provider names |
| `get_cache_stats( cacheName )` | Returns hit/miss/size statistics for a cache |
| `get_cache_keys( cacheName )` | Lists all keys currently in a cache |
| `get_cache_size( cacheName )` | Returns the number of objects in a cache |
| `get_cache_key_metadata( cacheName, objectKey )` | Returns metadata for a single cached entry |
| `cache_key_exists( cacheName, objectKey )` | Checks whether a key exists in a cache |
| `get_store_metadata_report( cacheName )` | Returns full store metadata for all entries |
| `clear_all( cacheName )` | Clears all entries from a cache |
| `clear_item( cacheName, objectKey )` | Clears a single entry by key |
| `clear_all_events( cacheName, [async] )` | Clears all event-cached entries |
| `clear_event( cacheName, eventSnippet, [queryString] )` | Clears a specific event cache entry |
| `clear_event_multi( cacheName, eventSnippets, [queryString] )` | Clears multiple event cache entries |
| `clear_all_views( cacheName, [async] )` | Clears all view-cached entries |
| `clear_view( cacheName, viewSnippet )` | Clears a specific view cache entry |
| `clear_view_multi( cacheName, viewSnippets )` | Clears multiple view cache entries |

### LogBox (`LogBoxTools`)

| Tool | Description |
|------|-------------|
| `get_logbox_info` | Returns LogBox version, ID, loggers, and appenders |
| `get_loggers` | Returns all registered logger definitions |
| `get_logger( category )` | Returns the logger definition for a category |
| `get_appenders` | Returns all registered appender definitions |
| `get_root_logger` | Returns the ROOT logger definition |
| `read_log_entries( [appenderName], [limit=100] )` | Reads the last N entries from file-based appenders |
| `get_last_error( [appenderName] )` | Finds the most recent ERROR or FATAL entry in log files |

### Interceptors (`InterceptorTools`)

| Tool | Description |
|------|-------------|
| `get_interceptors` | Returns all registered interceptors |
| `get_interceptor_states` | Returns all registered interception points and their listeners |

### Schedulers (`SchedulerTools`)

| Tool | Description |
|------|-------------|
| `get_schedulers` | Returns all registered application schedulers and their tasks |
| `get_scheduler( schedulerName )` | Returns metadata for a specific scheduler |
| `get_task_stats( schedulerName, taskName )` | Returns execution stats for a specific task |
| `run_task( schedulerName, taskName )` | Executes a scheduled task on demand |
| `pause_task( schedulerName, taskName )` | Pauses a scheduled task |
| `resume_task( schedulerName, taskName )` | Resumes a paused scheduled task |

### Async (`AsyncTools`)

| Tool | Description |
|------|-------------|
| `get_executors` | Returns all registered async executors with their stats |
| `get_executor_names` | Returns a flat list of executor names |

---

## Resources

`cbMCP` also exposes **MCP Resources** — ambient read-only context automatically injected into AI conversations:

| URI | Description |
|-----|-------------|
| `coldbox://app/settings` | All active ColdBox application configuration settings |
| `coldbox://app/modules` | All currently activated modules with key metadata |

---

## Learning ColdBox

ColdBox is the defacto standard for building modern ColdFusion (CFML) applications. It has the most extensive [documentation](https://coldbox.ortusbooks.com) of all modern web application frameworks.

If you don't like reading so much, then you can try our video learning platform: [CFCasts (www.cfcasts.com)](https://www.cfcasts.com)

## Ortus Sponsors

ColdBox is a professional open-source project and it is completely funded by the [community](https://patreon.com/ortussolutions) and [Ortus Solutions, Corp](https://www.ortussolutions.com). Ortus Patreons get many benefits like a cfcasts account, a FORGEBOX Pro account and so much more. If you are interested in becoming a sponsor, please visit our patronage page: [https://patreon.com/ortussolutions](https://patreon.com/ortussolutions)

### THE DAILY BREAD

> "I am the way, and the truth, and the life; no one comes to the Father, but by me (JESUS)" Jn 14:1-12
