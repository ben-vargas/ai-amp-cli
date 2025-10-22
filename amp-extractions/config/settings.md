# Amp CLI Settings Reference

Complete reference of all Amp CLI configuration settings, including both documented (public) and undocumented (internal/experimental/hidden) options.

**Configuration File**: `~/.config/amp/settings.json`

---

## Table of Contents

- [Documented Settings](#documented-settings)
  - [General](#general-documented)
  - [Git Integration](#git-integration)
  - [Tools & Execution](#tools--execution-documented)
  - [Terminal](#terminal-documented)
  - [MCP Integration](#mcp-integration)
  - [Editor Only](#editor-only-documented)
  - [CLI Only](#cli-only-documented)
- [Undocumented Settings](#undocumented-settings)
  - [Core](#core-undocumented)
  - [Anthropic](#anthropic-undocumented)
  - [Model Configuration](#model-configuration-undocumented)
  - [Experimental Agent Modes](#experimental-agent-modes)
  - [Terminal & Input](#terminal--input-undocumented)
  - [Tab Completion](#tab-completion-undocumented)
  - [Tools & Execution](#tools--execution-undocumented)
  - [Experimental Features](#experimental-features)
  - [Guidance Files](#guidance-files)
  - [Hooks System](#hooks-system)
  - [OpenRouter Integration](#openrouter-integration)
  - [JetBrains Integration](#jetbrains-integration)
  - [Deprecated Settings](#deprecated-settings)
- [Environment Variables](#environment-variables)

---

## Documented Settings

These settings are officially documented in the [Amp Manual](https://ampcode.com/manual#configuration).

### General (Documented)

#### `amp.anthropic.thinking.enabled`
- **Type**: `boolean`
- **Default**: `true`
- **Description**: Enable Claude's extended thinking process for deeper reasoning on complex tasks.
- **Source**: [Manual](https://ampcode.com/manual#configuration)

#### `amp.permissions`
- **Type**: `array` of permission objects
- **Default**: `[]`
- **Description**: Configure per-directory permissions for file access, bash commands, and MCP server usage.
- **Source**: [Manual](https://ampcode.com/manual#configuration)

**Example**:
```json
{
  "amp.permissions": [
    {
      "path": "/home/user/safe-project",
      "allow": ["bash", "edit"]
    },
    {
      "path": "/home/user/restricted",
      "deny": ["bash"]
    }
  ]
}
```

---

### Git Integration

#### `amp.git.commit.ampThread.enabled`
- **Type**: `boolean`
- **Default**: `true`
- **Description**: Include Amp thread ID in git commit messages for tracking.
- **Source**: [Manual](https://ampcode.com/manual#configuration)

#### `amp.git.commit.coauthor.enabled`
- **Type**: `boolean`
- **Default**: `true`
- **Description**: Add Amp as a co-author in git commit messages.
- **Source**: [Manual](https://ampcode.com/manual#configuration)

---

### Tools & Execution (Documented)

#### `amp.todos.enabled`
- **Type**: `boolean`
- **Default**: `true`
- **Description**: Enable TODO tracking and management features for multi-step tasks.
- **Source**: [Manual](https://ampcode.com/manual#configuration)

#### `amp.tools.disable`
- **Type**: `array` of strings
- **Default**: `[]`
- **Description**: Disable specific tools by name (e.g., `["Bash", "edit_file"]`).
- **Source**: [Manual](https://ampcode.com/manual#configuration)

#### `amp.tools.stopTimeout`
- **Type**: `number`
- **Default**: `300` (seconds)
- **Description**: Timeout for stopping long-running commands.
- **Source**: [Manual](https://ampcode.com/manual#configuration)

---

### Terminal (Documented)

#### `amp.terminal.commands.nodeSpawn.loadProfile`
- **Type**: `string`
- **Default**: `"always"`
- **Description**: Control when to load shell profile for bash commands. Options: `"always"`, `"never"`, `"auto"`.
- **Source**: [Manual](https://ampcode.com/manual#configuration)

---

### MCP Integration

#### `amp.mcpServers`
- **Type**: `object`
- **Default**: `{}`
- **Description**: Configure Model Context Protocol (MCP) servers for extending Amp with custom tools and resources.
- **Source**: [Manual](https://ampcode.com/manual#configuration)

**Example**:
```json
{
  "amp.mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed"],
      "alwaysAllow": ["read_file", "list_directory"]
    }
  }
}
```

---

### Editor Only (Documented)

These settings only apply when using Amp in VS Code or other editor integrations.

#### `amp.debugLogs`
- **Type**: `boolean`
- **Default**: `false`
- **Description**: Enable debug logging in the editor extension.
- **Source**: [Manual](https://ampcode.com/manual#configuration)

#### `amp.notifications.enabled`
- **Type**: `boolean`
- **Default**: `true`
- **Description**: Show in-editor notifications.
- **Source**: [Manual](https://ampcode.com/manual#configuration)

#### `amp.notifications.system.enabled`
- **Type**: `boolean`
- **Default**: `true`
- **Description**: Show system-level (OS) notifications.
- **Source**: [Manual](https://ampcode.com/manual#configuration)

#### `amp.tab.enabled`
- **Type**: `boolean`
- **Default**: `false`
- **Description**: Enable tab completion (inline code suggestions).
- **Source**: [Manual](https://ampcode.com/manual#configuration)

#### `amp.ui.zoomLevel`
- **Type**: `number`
- **Default**: `1`
- **Description**: UI zoom level (0.5 to 2.0).
- **Source**: [Manual](https://ampcode.com/manual#configuration)

---

### CLI Only (Documented)

#### `amp.updates.mode`
- **Type**: `string`
- **Default**: `"auto"`
- **Description**: Control automatic updates. Options: `"auto"`, `"manual"`, `"never"`.
- **Source**: [Manual](https://ampcode.com/manual#configuration)

---

## Undocumented Settings

These settings are internal, experimental, or hidden. They may change without notice.

> ⚠️ **Warning**: Use at your own risk. These settings are not officially supported.

---

### Core (Undocumented)

#### `amp.url`
- **Type**: `string`
- **Default**: `"https://ampcode.com"`
- **Visibility**: Hidden (internal)
- **Description**: The Amp server URL to connect to. All API requests are proxied through this URL.
- **Source**: `main.js:5653`

---

### Anthropic (Undocumented)

#### `amp.anthropic.interleavedThinking.enabled`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden (internal)
- **Description**: Enable interleaved thinking for Claude 4 models. Allows reasoning between tool calls rather than only at the beginning.
- **Source**: `main.js:5653`

#### `amp.anthropic.temperature`
- **Type**: `number`
- **Default**: `1.0`
- **Visibility**: Hidden (internal)
- **Description**: Temperature setting for Anthropic models (0.0 = deterministic, 1.0 = creative).
- **Note**: Only takes effect when thinking is disabled. Internal use only.
- **Source**: `main.js:5653`

#### `amp.anthropic.provider`
- **Type**: `string`
- **Default**: `"anthropic"`
- **Visibility**: Hidden (internal)
- **Description**: Which provider to use for Claude models. Options: `"anthropic"` or `"vertex"` (Google Cloud Vertex AI).
- **Source**: `main.js:5653`

---

### Model Configuration (Undocumented)

#### `amp.internal.model`
- **Type**: `string`
- **Default**: `undefined`
- **Visibility**: Hidden (internal)
- **Description**: Override the primary model. Format: `"provider:model"` (e.g., `"anthropic:claude-sonnet-4-5-20250929"`).

#### `amp.internal.oracleModel`
- **Type**: `string`
- **Default**: `"gpt-5"`
- **Visibility**: Hidden (internal)
- **Description**: Override which model the Oracle tool uses. Can be any OpenAI Responses API model (e.g., `"gpt-5-mini"`, `"o3"`, `"o3-mini"`).
- **Source**: `main.js:5653`

#### `amp.internal.primaryModel`
- **Type**: `string`
- **Default**: `undefined`
- **Visibility**: Hidden (DEPRECATED)
- **Description**: **[DEPRECATED]** Migrated to `amp.experimental.agentMode`. CLI auto-migrates on startup.

**Migration Examples**:
- `"anthropic/claude-opus-4-1-20250805"` → `"opus4.1"`
- `"openai/gpt-5"` → `"geppetto:main"`

#### `amp.internal.visibleModes`
- **Type**: `array` of strings
- **Default**: `[]`
- **Visibility**: Hidden (internal)
- **Description**: Array of agent mode keys to make visible in UI mode selector beyond defaults.

**Example**:
```json
{
  "amp.internal.visibleModes": ["fast", "geppetto:main", "bolt:main"]
}
```

#### `amp.internal.showCost`
- **Type**: `boolean`
- **Default**: `true`
- **Visibility**: Hidden (internal)
- **Description**: Display cost/credit information in the UI.

---

### Experimental Agent Modes

#### `amp.experimental.agentMode`
- **Type**: `string`
- **Default**: `undefined`
- **Visibility**: Hidden (internal)
- **Description**: Set experimental agent mode configuration. Overrides default mode selection.

**Production Modes**:
- `"smart"` - Claude Sonnet 4.5 (default)
- `"fast"` - Grok Code Fast 1
- `"free"` - Gemini 2.5 Flash Lite (free tier)

**Experimental Modes**:
- `"gronk-fast:main"` - Grok Code Fast 1
- `"gronk-fast:search"` - Claude main + Grok search
- `"gronk-fast:main+search"` - Grok for both
- `"bolt:main"` - Qwen 3 Coder 480B
- `"bolt:search"` - Claude main + Qwen search
- `"bolt:main+search"` - Qwen for both
- `"geppetto:main"` - GPT-5
- `"geppetto-pro:main"` - GPT-5 with high reasoning effort
- `"pinocchio:main"` - GPT-5 Codex
- `"claudius:main"` - Claude Opus 4
- `"opus4.1"` - Claude Opus 4.1
- `"hydrogen:main"` - Code Supernova
- `"kashmir:main"` - Kimi K2 Instruct (requires OpenRouter key)
- `"gossip:main"` - GPT OSS 120B
- `"sonomoon:main"` - Sonoma Sky Alpha (requires OpenRouter key)

Plus `:search` and `:main+search` variants for most modes.

---

### Terminal & Input (Undocumented)

#### `amp.submitOnEnter`
- **Type**: `boolean`
- **Default**: `true`
- **Visibility**: Hidden (internal)
- **Description**: Submit messages on Enter (true) or require Ctrl+Enter (false).

#### `amp.terminal.commands.nodeSpawn.detachedLogsDirectory`
- **Type**: `string`
- **Default**: `undefined`
- **Visibility**: Hidden (internal)
- **Description**: Directory to store detached command logs when using node-spawn mode.
- **Source**: `main.js:5653`

---

### Tab Completion (Undocumented)

> **Note**: These settings are primarily for VS Code extension integration.

#### `amp.tab.disabledLanguages`
- **Type**: `array` of strings
- **Default**: `[]`
- **Visibility**: Hidden
- **Description**: Language IDs for which tab completion is disabled (e.g., `["python", "javascript"]`).

#### `amp.tab.disableOnComment`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Disable tab completion when cursor is on a comment line.

#### `amp.tab.autoImport.enabled`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Enable automatic import statements for tab suggestions.

#### `amp.tab.experimental.unobtrusiveDiffs`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Experimental unobtrusive diffs mode to avoid layout shifts.

#### `amp.tab.experimental.showHotStreakProgress`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Show hot streak progress indicator.

#### `amp.tab.experimental.autoAccept.delayMs`
- **Type**: `number`
- **Default**: `0`
- **Visibility**: Hidden
- **Description**: Delay in milliseconds before auto-accepting suggestions. 0 = disabled.

#### `amp.tab.experimental.disableClientFilters`
- **Type**: `array` of strings
- **Default**: `[]`
- **Visibility**: Hidden
- **Description**: Disable specific client-side filters for debugging.

#### `amp.tab.diagnosticTracking`
- **Type**: `string`
- **Default**: `"multi_file"`
- **Visibility**: Hidden
- **Description**: Diagnostic tracking mode. Options: `"disabled"`, `"same_file"`, `"multi_file"`.

#### `amp.tab.hoverDiagnosticSuggestions`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Enable hover diagnostics for tab suggestions.

#### `amp.tab.experimental.displayMode`
- **Type**: `string`
- **Default**: `"eager"`
- **Visibility**: Hidden
- **Description**: Display mode. Options: `"eager"` (immediate), `"subtle"` (hint first).

#### `amp.tab.dismissCommandIds`
- **Type**: `array` of strings
- **Default**: `["extension.vim_escape", "vim.escape", "vscode-neovim.escape", "vscode-neovim.escapeKey"]`
- **Visibility**: Hidden
- **Description**: Command IDs to execute when escape is pressed (Vim compatibility).

#### `amp.tab.verboseLogs`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Enable verbose logging for tab completion debugging.

---

### Tools & Execution (Undocumented)

#### `amp.tools.inactivityTimeout`
- **Type**: `number`
- **Default**: `300` (seconds)
- **Visibility**: Hidden (internal)
- **Description**: Auto-cancel bash commands after this many seconds of no output.
- **Source**: `main.js:5653`

---

### Experimental Features

#### `amp.experimental.run_javascript.enabled`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Enable the experimental `run_javascript` tool for executing JavaScript code.

#### `amp.experimental.cli.nativeSecretsStorage.enabled`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Use native OS keychain/credential manager instead of `secrets.json`.
- **Source**: `main.js:5653`

#### `amp.experimental.reviewTool`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Enable automated code review tool using a subagent.

#### `amp.experimental.tools`
- **Type**: `array` of strings
- **Default**: `[]`
- **Visibility**: Hidden
- **Description**: Enable experimental tools by name. Available: `"chat_llm"`.

#### `amp.experimental.mcpPermissions`
- **Type**: `unknown`
- **Default**: `undefined`
- **Visibility**: Hidden (DEPRECATED)
- **Description**: **[DEPRECATED]** Auto-migrated to `amp.mcpPermissions`.

---

### Guidance Files

#### `amp.guidanceFiles.specs`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Read `*.spec.md` files adjacent to `AGENT.md`/`AGENTS.md` and include in context.

---

### Hooks System

#### `amp.hooks`
- **Type**: `array` of hook objects
- **Default**: `[]`
- **Visibility**: Hidden
- **Description**: Custom hooks for extending Amp functionality. Advanced programmatic customization.
- **Source**: `main.js:1941`

---

### OpenRouter Integration

#### `amp.openrouter.apiKey`
- **Type**: `string`
- **Default**: `undefined`
- **Visibility**: Not explicitly documented
- **Description**: API key for OpenRouter. Required for `kashmir:main` and `sonomoon:main` modes.
- **Alternative**: Can use `OPENROUTER_API_KEY` environment variable.

---

### JetBrains Integration

#### `amp.jetbrains.skipInstall`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Skip JetBrains plugin installation flow during startup.

---

### Deprecated Settings

#### `amp.gpt5`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden (DEPRECATED)
- **Description**: **[DEPRECATED]** GPT-5 is now the Oracle, not main model. See [GPT-5 Oracle announcement](https://ampcode.com/news/gpt-5-oracle).

---

## Environment Variables

### Documented Environment Variables

These are printed in CLI help output.

**Source**: `main.js:5680`

#### `AMP_API_KEY`
API key for authentication (alternative to interactive login).

#### `AMP_URL`
Override Amp server URL (default: `https://ampcode.com`).

#### `AMP_LOG_LEVEL`
Logging level. Options: `debug`, `info`, `warn`, `error`.

#### `AMP_LOG_FILE`
Path to log file for persistent logging.

#### `AMP_SETTINGS_FILE`
Custom path to settings.json file.

---

### Standard Node.js Variables

#### `HTTP_PROXY` / `HTTPS_PROXY`
HTTP/HTTPS proxy server URL.

#### `NODE_EXTRA_CA_CERTS`
Path to additional CA certificates.

---

### Undocumented Environment Variables

#### `AMP_CLI_STDOUT_DEBUG`
- **Type**: Flag (presence enables)
- **Default**: Not set
- **Description**: Force console debug transport for logs.
- **Source**: `main.js:5600`

#### `AMP_DEBUG`
- **Type**: Flag (presence enables)
- **Default**: Not set
- **Description**: Increase error output detail.
- **Source**: `main.js:5601`

#### `AMP_SKIP_UPDATE_CHECK`
- **Type**: Flag (presence enables)
- **Default**: Not set
- **Description**: Disable update check (overrides `amp.updates.mode`).
- **Source**: `main.js:5757-5760`

#### `AMP_HOME`
- **Type**: Path
- **Default**: Platform-specific
- **Description**: Install root / PATH wrapper behavior.
- **Source**: `main.js:5726-5746`

#### `AMP_VERSION`
- **Type**: Version string
- **Default**: Latest
- **Description**: Pin bootstrap version.
- **Source**: `main.js:5755`

#### `AMP_TEST_UPDATE_STATUS`
- **Type**: String
- **Default**: Not set
- **Description**: Test hook for updater state (internal testing).
- **Source**: `main.js:5757`

#### `OPENROUTER_API_KEY`
- **Type**: API key string
- **Default**: Not set
- **Description**: OpenRouter API key (alternative to `amp.openrouter.apiKey`).
- **Used for**: Kimi K2 and Sonoma Sky Alpha models.

---

### Toolbox Integration Variables

#### `AMP_TOOLBOX`
- **Type**: Directory path
- **Default**: Not set
- **Description**: Toolbox directory path for JetBrains integration.
- **Source**: `main.js:6069`

#### `TOOLBOX_ACTION`
- **Type**: Protocol string
- **Default**: Not set
- **Description**: Toolbox protocol handler for JetBrains.
- **Source**: `main.js:5891-5902`

---

## Configuration Examples

### Minimal Configuration
```json
{
  "amp.git.commit.coauthor.enabled": false
}
```

### Advanced Configuration
```json
{
  "amp.experimental.agentMode": "geppetto:main",
  "amp.internal.oracleModel": "gpt-5-mini",
  "amp.internal.visibleModes": ["fast", "geppetto:main", "bolt:main"],
  "amp.anthropic.thinking.enabled": true,
  "amp.tools.disable": ["run_javascript"],
  "amp.permissions": [
    {
      "path": "/home/user/safe-project",
      "allow": ["bash", "edit"]
    }
  ],
  "amp.mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/allowed/path"]
    }
  }
}
```

### OpenRouter Models
```json
{
  "amp.openrouter.apiKey": "sk-or-v1-...",
  "amp.experimental.agentMode": "sonomoon:main"
}
```

### Native Secrets Storage
```json
{
  "amp.experimental.cli.nativeSecretsStorage.enabled": true
}
```

---

## Notes on Settings

### Visibility Flags
- **Documented**: Officially supported, documented in manual
- **Hidden**: Internal use, not in manual, may change
- **Experimental**: Preview features, may change or be removed
- **Deprecated**: Legacy settings, will be removed

### Setting Scopes
- **CLI Only**: Only affects command-line interface
- **Editor Only**: Only affects VS Code/editor integration
- **Both**: Affects both CLI and editor

### Migration Warnings
- `amp.internal.primaryModel` → Use `amp.experimental.agentMode` instead
- `amp.experimental.mcpPermissions` → Migrated to `amp.mcpPermissions`
- `amp.gpt5` → GPT-5 is now Oracle only

---

## Related Resources

- [Official Amp Manual](https://ampcode.com/manual)
- [Amp Configuration Guide](https://ampcode.com/manual#configuration)
- [Agent Modes Documentation](https://ampcode.com/news/gpt-5-oracle)
- [Model Endpoints Reference](./endpoints.md)

---

**Generated from**: Amp CLI v0.0.1761076893-ge5520f source code analysis  
**Verified by**: Oracle analysis of `node_modules/@sourcegraph/amp/dist/main.js`  
**Settings Registry Source**: `main.js:5653` (CQ6)  
**Last Updated**: 2025-01-20
