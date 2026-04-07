---
description: >-
  Use the Developer MCP Server as a CLI for debugging, introspection, and
  advanced configuration.
---

# CLI Usage

The Developer MCP Server is designed to run as an MCP server connected to your AI host. You do not need the CLI for normal usage.

The CLI is useful when you want to inspect available tools, debug your configuration, or test tools outside of an MCP session. All MCP servers built on the Base MCP SDK share the same CLI interface.

## Claude Code Plugin

If you are using Claude Code, install the `umbraco-mcp-server` plugin. The `/mcp-cli` skill guides you interactively through configuration, filtering, and troubleshooting.

```bash
/plugin marketplace add umbraco/Umbraco-MCP-Base
/plugin install umbraco-mcp-server@umbraco/Umbraco-MCP-Base
```

Once installed, run `/mcp-cli` for interactive help.

## CLI Reference

For the full reference of CLI flags, runtime modes (readonly and dry-run), introspection commands, and input sanitization, see the [CLI Reference](https://app.gitbook.com/s/qRBjeReNuznLmI2zTKUq/sdk/cli).
