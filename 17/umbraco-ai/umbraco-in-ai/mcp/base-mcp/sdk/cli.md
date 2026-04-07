---
description: >-
  CLI reference for running Umbraco MCP servers built with the SDK, including
  authentication, runtime modes, tool filtering, and introspection commands.
---

# CLI Reference

Umbraco MCP servers built with `@umbraco-cms/mcp-server-sdk` run as CLI tools over stdio. The CLI handles authentication and configuration, then exposes tools that communicate with the Umbraco Management API. Your AI agent (Claude Code, Cursor, and others) calls these tools to read and manage content in your Umbraco instance.

## Claude Code Plugin

If you are using Claude Code, install the `umbraco-mcp-server` plugin. The plugin provides a `/mcp-cli` skill that guides you interactively through setup, configuration, filtering, and debugging. This is the recommended way to work with the CLI.

To install the plugin, run the following commands in Claude Code:

```bash
/plugin marketplace add umbraco/Umbraco-MCP-Base
/plugin install umbraco-mcp-server@umbraco/Umbraco-MCP-Base
```

Once installed, run `/mcp-cli` at any time for guided help. The skill understands your current configuration and walks you through each step.

The sections below provide the full reference for all CLI options. You do not need to read them when using the plugin.

{% hint style="info" %}
The CLI is designed to be consumed by AI agents, not operated directly by humans. You configure the CLI with environment variables or flags, then your AI agent connects and interacts with Umbraco through the exposed tools. The introspection commands (`--list-tools`, `--debug-config`, and others) are the human-facing part. Use them to understand and verify what your agent sees.
{% endhint %}

## Authentication and Configuration

The CLI requires three environment variables for authentication: `UMBRACO_CLIENT_ID`, `UMBRACO_CLIENT_SECRET`, and `UMBRACO_BASE_URL`. Create these credentials in the Umbraco backoffice under **Settings > Users** as an API user.

CLI arguments take precedence over environment variables, which take precedence over `.env` file values. A `.env` file in the current working directory is loaded automatically. Use `--env /path/to/.env` to specify a custom location.

For the full list of configuration fields, precedence rules, and custom field definitions, see [Configuration](configuration.md).

## Starting the Server

```bash
# Via .env file (recommended) — create .env with credentials, then:
node dist/index.js

# Via npx with .env file
npx @umbraco-cms/mcp-dev

# Via custom .env path
node dist/index.js --env /path/to/.env
```

### Claude Code Configuration

Use the `env` block for credentials. These are passed as environment variables to the process, not as CLI arguments:

```json
{
  "mcpServers": {
    "umbraco": {
      "command": "npx",
      "args": ["@umbraco-cms/mcp-dev"],
      "env": {
        "UMBRACO_CLIENT_ID": "your-client-id",
        "UMBRACO_CLIENT_SECRET": "your-secret",
        "UMBRACO_BASE_URL": "https://localhost:44391"
      }
    }
  }
}
```

## Tool Filtering

You can control which tools are exposed to the LLM using modes, collections, slices, and individual tool names. All filters accept comma-separated values via CLI flags or environment variables.

```bash
# Read-only content browsing
UMBRACO_INCLUDE_SLICES=read,list,search \
UMBRACO_INCLUDE_TOOL_COLLECTIONS=content,media \
node dist/index.js

# Everything except delete operations
UMBRACO_EXCLUDE_SLICES=delete node dist/index.js

# Only specific tools
UMBRACO_INCLUDE_TOOLS=get-content-by-id,list-content node dist/index.js
```

For the full list of filter flags, available slices, and precedence rules, see [Tool Filtering](tool-filtering.md).

## Runtime Modes

### Readonly Mode

```bash
node dist/index.js --umbraco-readonly
# or: UMBRACO_READONLY=true
```

Mutation tools are removed from the server. The agent cannot see or call them. Only tools with `readOnlyHint: true` are registered. Use this mode when you want zero risk of data modification.

### Dry-Run Mode

```bash
node dist/index.js --umbraco-dry-run
# or: UMBRACO_DRY_RUN=true
```

Read-only tools execute normally and return real data. Mutation tools return a structured preview without calling the Umbraco API:

```json
{
  "dryRun": true,
  "toolName": "delete-example",
  "wouldExecute": true,
  "inputReceived": { "id": "550e8400-e29b-41d4-a716-446655440000" },
  "annotations": { "readOnlyHint": false, "destructiveHint": true }
}
```

Input validation still runs, so the LLM receives validation feedback. Use dry-run mode for safe exploration. The LLM can try mutation tools without risk.

### Readonly vs Dry-Run

| | Readonly | Dry-Run |
|---|---------|---------|
| LLM sees mutation tools | No | Yes |
| Mutation tools execute | N/A | No (preview only) |
| Read tools execute | Yes | Yes |
| Risk level | Zero | Minimal |

## Introspection Commands

These flags print output and exit immediately. They do not start the MCP server and do not require auth credentials or a running Umbraco instance.

Introspection respects all filtering configuration. If you set `UMBRACO_READONLY=true` or any filtering env var, the output shows only tools that pass those filters. This matches what the LLM sees at runtime.

| Flag | Description |
|------|-------------|
| `--list-tools` | Print ASCII table of all tools (name, collection, slices, annotations) |
| `--describe-tool <name>` | Print full JSON schema and metadata for a specific tool (exits 1 if not found or filtered out) |
| `--call <name>` | Call a tool by name, print the result as JSON, and exit |
| `--call-args <json>` | JSON arguments for `--call` (default: `{}`) |
| `--generate-context` | Output structured CONTEXT.md documenting all tools (pipe to file) |
| `--debug-config` | Print resolved configuration as JSON (values, sources, filter config) |

### `--list-tools` Output

```
Name             | Collection | Slices | RO | Destr | Description
-----------------+------------+--------+----+-------+---------------------------------------------
get-example      | example    | read   | Y  | N     | Gets an example item by ID.
list-examples    | example    | list   | Y  | N     | Lists all example items with pagination.
create-example   | example    | create | N  | N     | Creates a new example item.
delete-example   | example    | delete | N  | Y     | Deletes an example item by ID.
```

### `--describe-tool` Output

```json
{
  "name": "get-example",
  "collection": "example",
  "description": "Gets an example item by ID.",
  "slices": ["read"],
  "annotations": { "readOnlyHint": true },
  "inputSchema": {
    "type": "object",
    "properties": {
      "id": { "type": "string", "description": "The example item ID (UUID)" }
    },
    "required": ["id"]
  }
}
```

### Examples

```bash
# See all tools
node dist/index.js --list-tools

# See only what the LLM sees with filtering
UMBRACO_READONLY=true node dist/index.js --list-tools
UMBRACO_INCLUDE_SLICES=read,list node dist/index.js --list-tools

# Get schema for a specific tool
node dist/index.js --describe-tool get-content-by-id

# Call a tool directly and print the result
node dist/index.js --call get-content-by-id --call-args '{"id": "550e8400-e29b-41d4-a716-446655440000"}'

# Generate documentation
node dist/index.js --generate-context > CONTEXT.md

# Debug configuration — see resolved values and their sources
node dist/index.js --debug-config
UMBRACO_READONLY=true UMBRACO_INCLUDE_SLICES=read node dist/index.js --debug-config
```

### Debug Config Output

`--debug-config` prints JSON showing every config field with its resolved value and source (`cli`, `env`, or `none`). Credentials are masked. The `resolvedFilterConfig` section shows the final filter state applied to tools.

```json
{
  "envFile": { "source": "default" },
  "auth": {
    "baseUrl": { "value": "https://localhost:44391", "source": "env" },
    "clientId": { "value": "***", "source": "env" },
    "clientSecret": { "value": "***", "source": "env" }
  },
  "filtering": {
    "readonly": { "value": true, "source": "env" },
    "includeSlices": { "value": ["read", "list"], "source": "env" }
  },
  "resolvedFilterConfig": {
    "readOnly": true,
    "enabledSlices": ["read", "list"]
  }
}
```

## Other Options

| Flag | Env Var | Description |
|------|---------|-------------|
| `--umbraco-allowed-media-paths` | `UMBRACO_ALLOWED_MEDIA_PATHS` | Restrict media operations to these paths |
| `--disable-output-compatibility-mode` | `DISABLE_OUTPUT_COMPATIBILITY_MODE` | Use structured output instead of text |

## Input Sanitization

The SDK validates all string inputs before tool handlers run:

* Rejects control characters, path traversal (`../`), embedded query params, and percent-encoded strings
* Validates UUID format where expected
* Returns ProblemDetails (RFC 7807) with clear error messages

The LLM receives validation errors and can self-correct. No configuration is needed.
