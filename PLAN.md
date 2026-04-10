# cbAITools - ColdBox MCP Server Module

## Project Overview

`cbAITools` is a BoxLang PRIME module that exposes ColdBox applications to AI agents via the Model Context Protocol (MCP). This module enables AI assistants (like Claude, Cursor, VS Code Copilot, etc.) to introspect, understand, and interact with ColdBox applications in real-time.

Inspired by Laravel Boost, this module aims to be the **essential development companion** for every ColdBox developer, providing comprehensive insight into application structure, configuration, and runtime behavior.

## Core Architecture

### MCP HTTP Server Integration

The module leverages **BoxLang AI** ([ai.ortusbooks.com](https://ai.ortusbooks.com)) and specifically the MCP capabilities from the `bxai` module.

**Entry Point**: `/mcp` route handler that bootstraps the MCP HTTP server:

```boxlang
// config/Router.bx
route( "/mcp" ).to( ( event, rc, prc ) => {
    import bxModules.bxai.models.mcp.MCPRequestProcessor
    MCPRequestProcessor::startHttp()
} )
```

This creates a persistent HTTP endpoint that AI clients can connect to using the MCP protocol.

### Security Model

**Development Mode (Default)**:
- No authentication required
- Enabled automatically in development environments
- Provides unrestricted access to introspection tools

**Production Mode (Optional)**:
- Requires `moduleSettings.security.enabled = true`
- API key authentication via `moduleSettings.security.apiKey`
- Validates requests using `Authorization: Bearer <apiKey>` header
- Recommended for production environments where MCP access is needed

## Tools Implementation

### Current Tools (Implemented in ToolsService.bx)

✅ **Application Introspection**
- `get_application_structure` - Full directory structure scan
- `get_coldbox_settings` - Framework configuration
- `time_now` - Server datetime

✅ **Routing**
- `list_routes` - HTTP routes registry
- `list_module_routes` - Module routing tables
- `get_module_routes` - Module-specific routes

✅ **Modules**
- `get_modules` - Detailed module registry (config, paths, status)
- `get_loaded_modules` - Simple list of active modules

✅ **Handlers (Controllers)**
- `get_handlers` - All handlers with actions/methods metadata

✅ **Async & Scheduling**
- `get_executors` - Async executor pools with thread statistics
- `get_schedulers` - Task schedulers with scheduled task details

✅ **Event System**
- `get_interceptors` - All interceptors by interception points

✅ **Caching**
- `get_caches` - CacheBox providers with statistics

✅ **Dependency Injection**
- `get_wirebox_mappings` - All DI container mappings

### Planned Tools (To Surpass Laravel Boost)

#### 🎯 Configuration & Environment
- [ ] `get_config` - Retrieve config values using dot notation (`database.default`)
- [ ] `list_config_keys` - All available configuration keys
- [ ] `list_env_vars` - Environment variables (with security filtering)
- [ ] `get_env` - Get specific environment variable value

#### 🗄️ Database & ORM
- [ ] `list_database_connections` - Available datasource connections
- [ ] `get_default_database` - Default datasource name
- [ ] `execute_query` - Run ad-hoc SQL queries (with safety limits)
- [ ] `get_database_schema` - Table/column structure for datasources
- [ ] `list_orm_entities` - All ORM entity mappings
- [ ] `get_entity_schema` - Specific entity properties/relationships
- [ ] `list_orm_relationships` - Entity relationship graph

#### 📝 Logging & Debugging
- [ ] `read_log_entries` - Tail last N log entries from LogBox appenders
- [ ] `get_last_error` - Most recent application error
- [ ] `list_log_appenders` - All LogBox appenders and their configurations
- [ ] `get_log_levels` - Current logging levels by category
- [ ] `search_logs` - Search logs by keyword/pattern

#### 🔍 Code Introspection
- [ ] `list_models` - All registered models in WireBox
- [ ] `get_model_metadata` - Detailed model/service metadata
- [ ] `list_views` - All view templates
- [ ] `list_layouts` - All layout templates
- [ ] `search_handlers` - Find handlers by name pattern
- [ ] `get_handler_metadata` - Detailed handler action signatures

#### 🧪 Testing
- [ ] `list_test_specs` - All TestBox spec files
- [ ] `run_test_suite` - Execute tests and return results
- [ ] `get_test_coverage` - Code coverage report

#### 🔐 Security & Validation
- [ ] `list_validation_rules` - cbValidation rules
- [ ] `validate_data` - Validate data against constraints
- [ ] `list_security_rules` - cbSecurity rules and permissions

#### 📦 REST APIs
- [ ] `list_rest_handlers` - REST API handlers (if using cbRest/Relax)
- [ ] `get_rest_routes` - REST route definitions
- [ ] `test_rest_endpoint` - Make test HTTP request to endpoint

#### ⚡ Performance
- [ ] `get_memory_usage` - JVM heap/memory statistics
- [ ] `get_request_metrics` - Current request statistics
- [ ] `get_cache_statistics` - Detailed cache hit/miss rates
- [ ] `get_thread_dump` - JVM thread dump for debugging

#### 📊 Events & Messages
- [ ] `announce_event` - Trigger a ColdBox event
- [ ] `list_async_tasks` - Running async tasks
- [ ] `get_task_status` - Check async task status

#### 🛠️ Development Tools
- [ ] `execute_coldbox_command` - Run CommandBox tasks
- [ ] `reinit_framework` - Trigger framework reinit
- [ ] `clear_cache` - Clear specific or all caches
- [ ] `clear_view_cache` - Clear rendered view cache

#### 🌐 HTTP & Routing
- [ ] `get_absolute_url` - Convert relative paths to absolute URLs
- [ ] `test_route` - Simulate route resolution
- [ ] `match_route` - Find which route matches a URL

## Resources Implementation

MCP Resources provide read-only data that AI agents can reference:

### Planned Resources

- [ ] **Application Architecture Diagram** (`coldbox://architecture`)
  - Visual/textual representation of app structure
  - Modules, handlers, models hierarchy

- [ ] **Module Dependency Graph** (`coldbox://module-dependencies`)
  - Module loading order and dependencies

- [ ] **Configuration Reference** (`coldbox://config/{key}`)
  - Full config file contents with documentation

- [ ] **Log Streams** (`coldbox://logs/{appender}`)
  - Real-time log tailing as resources

- [ ] **Database ERD** (`coldbox://database/erd`)
  - Entity-relationship diagram in text format

- [ ] **API Documentation** (`coldbox://api/docs`)
  - Auto-generated API documentation from handlers

## Prompts Implementation

MCP Prompts provide reusable prompt templates for common tasks:

### Planned Prompts

- [ ] **Debug Error** - Help debug the last application error
- [ ] **Create Handler** - Generate a new handler with actions
- [ ] **Create Model** - Generate a model with validation
- [ ] **Write Test** - Generate TestBox spec for a handler
- [ ] **Optimize Query** - Analyze and optimize database query
- [ ] **Security Audit** - Review security configurations
- [ ] **Performance Analysis** - Analyze app performance bottlenecks
- [ ] **Generate Documentation** - Document handler/model
- [ ] **Create Migration** - Generate database migration script
- [ ] **Refactor Code** - Suggest refactoring for better patterns

## Module Configuration

### ModuleConfig.bx Structure

```boxlang
class {
    this.title = "cbAITools"
    this.author = "Ortus Solutions"
    this.description = "MCP Server for ColdBox Applications"
    this.version = "@build.version@+@build.number@"
    this.modelNamespace = "cbAITools"
    this.cfmapping = "cbAITools"
    this.dependencies = [ "bxai" ]
    
    function configure(){
        settings = {
            // Security settings
            security : {
                enabled : getSystemSetting( "CBAITOOLS_SECURITY", false ),
                apiKey : getSystemSetting( "CBAITOOLS_API_KEY", "" )
            },
            
            // MCP Server settings
            mcp : {
                serverName : "coldbox-mcp-server",
                serverVersion : this.version,
                enableTools : true,
                enableResources : true,
                enablePrompts : true
            },
            
            // Tool-specific settings
            tools : {
                maxLogEntries : 100,
                maxQueryRows : 1000,
                allowedLogAppenders : [],  // Empty = all
                databaseQueryTimeout : 30,  // seconds
                enableDangerousTools : false  // execute queries, reinit, etc.
            }
        }
    }
    
    function onLoad(){
        // Register MCP route
        if( settings.mcp.enableTools ){
            wirebox.getInstance( "router@coldbox" )
                .addRoute( 
                    pattern = "/mcp",
                    handler = "MCPHandler",
                    action = "index"
                )
        }
    }
}
```

## File Structure

```
cbAITools/
├── ModuleConfig.bx           # Module configuration
├── box.json                  # Module metadata
├── PLAN.md                   # This file
├── config/
│   └── Router.bx            # MCP route definitions
├── handlers/
│   └── MCPHandler.bx        # MCP HTTP endpoint handler
├── models/
│   ├── ToolsService.bx      # Tool implementations
│   ├── ResourcesService.bx  # Resource implementations (planned)
│   ├── PromptsService.bx    # Prompt implementations (planned)
│   └── SecurityService.bx   # API key validation (planned)
├── interceptors/
│   └── SecurityInterceptor.bx  # Request authentication (planned)
└── test-harness/
    └── tests/specs/
        ├── ToolsServiceSpec.bx
        ├── ResourcesServiceSpec.bx
        └── PromptsServiceSpec.bx
```

## Implementation Phases

### Phase 1: Core MCP Server (Current)
- ✅ Basic ToolsService with introspection tools
- ✅ Application structure tools
- ✅ Routing, modules, handlers tools
- ✅ Async, caching, DI tools

### Phase 2: Enhanced Tools
- [ ] Configuration & environment tools
- [ ] Database & ORM tools
- [ ] Logging & debugging tools
- [ ] Code introspection tools

### Phase 3: Resources & Prompts
- [ ] ResourcesService implementation
- [ ] Architecture and dependency resources
- [ ] PromptsService implementation
- [ ] Common prompt templates

### Phase 4: Security & Production
- [ ] SecurityService implementation
- [ ] API key authentication
- [ ] SecurityInterceptor for request validation
- [ ] Production-ready configurations

### Phase 5: Advanced Features
- [ ] Testing tools
- [ ] REST API tools
- [ ] Performance monitoring tools
- [ ] Development utility tools

## Usage Examples

### For AI Agents (Claude Desktop, etc.)

**MCP Configuration**:
```json
{
  "mcpServers": {
    "coldbox-app": {
      "type": "http",
      "url": "http://localhost:8080/mcp",
      "headers": {
        "Authorization": "Bearer your-api-key-here"
      }
    }
  }
}
```

### For Developers

**Development** (no auth):
```bash
# Start app
box server start

# AI agent connects to http://localhost:8080/mcp
# No authentication required
```

**Production** (with auth):
```bash
# Set environment variables
export CBAITOOLS_SECURITY=true
export CBAITOOLS_API_KEY="super-secret-key-12345"

# Start app
box server start

# AI agent must include: Authorization: Bearer super-secret-key-12345
```

## Advantages Over Laravel Boost

1. **More Comprehensive Module System**
   - ColdBox modules are first-class citizens
   - Hierarchical module support
   - Module-specific routing, configs, models

2. **Built-in Async & Scheduling**
   - Executor pool monitoring
   - Task scheduler introspection
   - Real-time task status

3. **Advanced Caching**
   - Multiple cache providers
   - Cache statistics and performance metrics
   - Distributed cache support

4. **Event-Driven Architecture**
   - Interceptor introspection by point
   - Event announcement capabilities
   - Complete event lifecycle visibility

5. **Powerful DI Container**
   - WireBox mapping introspection
   - Scope management
   - Virtual inheritance and delegation

6. **ORM Excellence**
   - If using cborm/Hibernate
   - Entity relationships
   - Lazy loading inspection

7. **Security Framework**
   - cbSecurity integration
   - Rule-based access control
   - Authentication/authorization introspection

## Testing Strategy

### Unit Tests
- Test each tool independently
- Mock ColdBox services
- Verify JSON serialization

### Integration Tests
- Test with real ColdBox application (test-harness)
- Test MCP HTTP endpoint
- Test authentication flow

### MCP Client Tests
- Test with actual AI clients
- Verify tool discovery
- Verify tool execution

## Documentation Requirements

1. **README.md** - Installation, basic usage, quick start
2. **SECURITY.md** - Security best practices, authentication setup
3. **TOOLS.md** - Complete tool reference with examples
4. **RESOURCES.md** - Resource URI reference
5. **PROMPTS.md** - Prompt template reference
6. **API.md** - MCP protocol details for this implementation

## Success Metrics

- ✅ 20+ introspection tools implemented
- ✅ Sub-100ms average tool response time
- ✅ 100% test coverage for tools
- ✅ Works with Claude Desktop, VS Code, Cursor
- ✅ Production-ready security
- ✅ Comprehensive documentation
- ✅ Community adoption by ColdBox developers

## Future Enhancements

- **Multi-App Support** - Connect to multiple ColdBox apps
- **Remote Debugging** - Attach to running production apps
- **Time-Travel Debugging** - Request replay and analysis
- **Auto-Fix Suggestions** - AI-powered code fixes
- **Performance Profiling** - Detailed performance analysis
- **Code Generation** - Template-based code generation
- **Migration Tools** - Framework upgrade assistance

---

**Goal**: Make cbAITools the **must-have** development tool for every ColdBox developer, providing unprecedented insight and control over ColdBox applications through AI-powered introspection.
