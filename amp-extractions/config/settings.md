# AMP CLI Settings Reference

**Version:** 0.0.1761153678-gfa55cf  
**Source:** `node_modules/@sourcegraph/amp/dist/main.js`  
**Last Updated:** 2025-01-21  
**Verification:** Oracle-assisted analysis

⚠️ **IMPORTANT:** See [Settings Verification Status](./settings-verification-status.md) for detailed verification of which settings are confirmed to work. Some settings in this document are unverified or may have discrepancies between registry defaults and runtime behavior.

---

## Table of Contents

1. [Overview](#overview)
2. [Documented Settings](#documented-settings)
3. [Undocumented Settings](#undocumented-settings)
4. [Environment Variables](#environment-variables)
5. [Configuration Examples](#configuration-examples)
6. [Notes on Settings](#notes-on-settings)
7. [Related Resources](#related-resources)
8. [Verification Status](#verification-status)

---

## Overview

This document provides a comprehensive reference of ALL Amp CLI configuration settings, including both documented and undocumented options. Settings are divided into:

- **Documented Settings**: Officially documented in the [Amp Manual](https://ampcode.com/manual#configuration)
- **Undocumented Settings**: Present in the code but not in official documentation (hidden, experimental, or internal)

### Configuration File

Settings are stored in a JSON or JSONC file, typically at:
- `~/.amp/settings.json` (default)
- Custom path via `AMP_SETTINGS_FILE` environment variable

### Setting Keys

All settings use the `amp.` prefix when stored in the configuration file:

```json
{
  "amp.notifications.enabled": true,
  "amp.anthropic.thinking.enabled": false
}
```

The prefix is stripped when the CLI loads settings internally.

---

## Documented Settings

These settings are officially documented in the [Amp Manual](https://ampcode.com/manual#configuration).

### General Settings

#### `amp.permissions`

**Type:** Array of permission rules  
**Default:** `[]`  
**Source:** Manual, Code: main.js:1244-1290  

**Description:** Define tool execution permission rules for security control.

**Permission Rule Structure:**
```json
{
  "tool": "tool-name",           // Tool to match (or "all")
  "action": "allow|ask|reject",  // Permission action
  "reason": "Explanation"        // Optional reason
}
```

**Examples:**
```json
{
  "amp.permissions": [
    {
      "tool": "Bash",
      "action": "ask",
      "reason": "Always confirm before running shell commands"
    },
    {
      "tool": "edit_file",
      "action": "allow"
    }
  ]
}
```

**Notes:**
- Merged with built-in permission rules
- Used to gate tool invocations
- More specific rules override general rules

---

#### `amp.notifications.enabled`

**Type:** Boolean  
**Default:** `true`  
**Source:** Manual, Code: main.js:5653 (CQ6 registry)  
**Visibility:** Visible

**Description:** Enable/disable completion and blocked notification sounds in the CLI UI.

**Example:**
```json
{
  "amp.notifications.enabled": false
}
```

---

#### `amp.notifications.system.enabled`

**Type:** Boolean  
**Default:** `true`  
**Source:** Manual, Code: main.js:5653 (CQ6 registry)  
**Visibility:** Visible

**Description:** Show system desktop notifications when terminal is not focused.

**Example:**
```json
{
  "amp.notifications.system.enabled": false
}
```

---

#### `amp.anthropic.thinking.enabled`

**Type:** Boolean  
**Default:** `true` (runtime default when unset)  
**Source:** Manual, Code: main.js:5653 (registry), main.js:1225 (runtime)  
**Visibility:** Hidden in code

**Description:** Enable Claude's extended thinking mode for deeper reasoning.

**Example:**
```json
{
  "amp.anthropic.thinking.enabled": false
}
```

**How It Works:**
- **When NOT set** (default): Thinking is ENABLED (`true`) via runtime fallback (`?? true` at line 1225)
- **When set to `true`**: Thinking is ENABLED (explicit)
- **When set to `false`**: Thinking is DISABLED (explicit)

**What You'll See:**
- **Enabled**: Claude shows thinking summaries in CLI output and web interface, uses extended reasoning tokens
- **Disabled**: No thinking blocks, faster responses, uses temperature setting instead

**Note:** The registry default (`false` at line 5653) is overridden by runtime logic that defaults to `true` when the setting is unset. The manual is correct - the effective default is `true`.

---

#### `amp.updates.mode`

**Type:** String enum: `"auto"` | `"warn"` | `"disabled"`  
**Default:** `"auto"`  
**Source:** Manual, Code: main.js:5757-5769  

**Description:** Control update checking and installation behavior.

**Values:**
- `auto` - Automatically check and install updates
- `warn` - Check for updates but only warn, don't auto-install
- `disabled` - Disable update checks entirely

**Example:**
```json
{
  "amp.updates.mode": "warn"
}
```

---

### Git Integration Settings

#### `amp.git.commit.ampThread.enabled`

**Type:** Boolean  
**Default:** `true`  
**Source:** Manual, Code: main.js:1337-1346  

**Description:** Add `Amp-Thread-ID` trailer to git commits.

**Trailer Format:**
```
Amp-Thread-ID: https://ampcode.com/threads/{threadId}
```

**Example:**
```json
{
  "amp.git.commit.ampThread.enabled": true
}
```

**Git Commit Example:**
```
feat: Add new feature

Implemented XYZ functionality

Amp-Thread-ID: https://ampcode.com/threads/abc123
```

---

#### `amp.git.commit.coauthor.enabled`

**Type:** Boolean  
**Default:** `true`  
**Source:** Manual, Code: main.js:1337-1346  

**Description:** Add `Co-authored-by` trailer for Amp to git commits.

**Trailer Format:**
```
Co-authored-by: Amp <amp@ampcode.com>
```

**Example:**
```json
{
  "amp.git.commit.coauthor.enabled": true
}
```

**Git Commit Example:**
```
feat: Add new feature

Implemented XYZ functionality

Co-authored-by: Amp <amp@ampcode.com>
```

---

### Tools & Execution Settings

#### `amp.todos.enabled`

**Type:** Boolean  
**Default:** `true`  
**Source:** Manual, Code: main.js:5653 (CQ6 registry)  
**Visibility:** Hidden

⚠️ **VERIFICATION WARNING:** This setting may be non-functional. Runtime code appears to hardcode `enableTodos:true` (line 1234) instead of reading the setting. See [verification status](./settings-verification-status.md#amp-todos-enabled) for details.

**Description:** Enable TODO tracking tool integration.

**Example:**
```json
{
  "amp.todos.enabled": false
}
```

**Notes:**
- Toggles TODO tracking tool in system prompt
- Affects whether agent can create/manage task lists
- **Warning:** Setting may be ignored by runtime code (unverified)

---

#### `amp.tools.disable`

**Type:** Array of strings  
**Default:** `["browser_navigate", "builtin:edit_file"]`  
**Source:** Manual, Code: main.js:5653 (CQ6 registry)  
**Visibility:** Visible

⚠️ **VERIFICATION WARNING:** Runtime usage not confirmed. Setting exists in registry but exact filtering logic not found. May need manual testing to verify functionality.

**Description:** Disable specific tools by name. Use `"builtin:toolname"` to disable only built-in tools.

**Example:**
```json
{
  "amp.tools.disable": [
    "browser_navigate",
    "builtin:edit_file",
    "Bash"
  ]
}
```

**Notes:**
- Completely removes tools from available tool list
- Different from permissions (which gate execution)
- `builtin:` prefix targets only built-in tools, not MCP tools
- **Verification status:** Unverified - needs manual testing

---

#### `amp.tools.stopTimeout`

**Type:** Number (seconds)  
**Default:** `300`  
**Source:** Manual  

⚠️ **VERIFICATION WARNING:** Not found in settings registry (line 5653). May be documented name for `amp.tools.inactivityTimeout` or may not be functional. Needs manual testing.

**Description:** Timeout for stopping long-running tools.

**Example:**
```json
{
  "amp.tools.stopTimeout": 600
}
```

**Note:** See also `amp.tools.inactivityTimeout` in undocumented settings (may be the same feature or this setting may not actually exist).

---

### Terminal Settings

#### `amp.terminal.commands.nodeSpawn.loadProfile`

**Type:** String enum: `"always"` | `"never"` | `"daily"`  
**Default:** `"always"`  
**Source:** Manual, Code: main.js:1333-1347  

**Description:** Control when to load shell profile environment for child processes (including MCP servers).

**Values:**
- `always` - Load profile for every command
- `never` - Never load profile
- `daily` - Load profile once per day

**Example:**
```json
{
  "amp.terminal.commands.nodeSpawn.loadProfile": "daily"
}
```

**Notes:**
- Affects environment variables available to spawned processes
- Can impact MCP server initialization
- Daily mode caches environment to avoid repeated shell startup overhead

---

### MCP Integration Settings

#### `amp.mcpServers`

**Type:** Object (server configuration map)  
**Default:** Example filesystem server  
**Source:** Manual, Code: main.js:5653 (CQ6 registry)  
**Visibility:** Visible

**Description:** Configure Model Context Protocol (MCP) servers to provide additional tools.

**Structure:**
```json
{
  "amp.mcpServers": {
    "server-name": {
      "command": "executable",
      "args": ["arg1", "arg2"],
      "env": {
        "KEY": "value"
      }
    }
  }
}
```

**Example:**
```json
{
  "amp.mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "@modelcontextprotocol/server-filesystem",
        "/home/user/projects"
      ]
    },
    "database": {
      "command": "node",
      "args": ["/path/to/db-mcp-server.js"],
      "env": {
        "DB_CONNECTION": "postgresql://..."
      }
    }
  }
}
```

**Notes:**
- Each server exposes tools to the agent
- Servers are spawned as child processes
- Environment variables can be passed to servers
- MCP specification: https://modelcontextprotocol.io

---

### Editor Only Settings

These settings apply only to the VS Code extension, not the CLI:

#### `amp.tab.enabled`

**Type:** Boolean  
**Default:** `false`  
**Source:** Manual  
**Scope:** Editor only

**Description:** Enable Tab completion engine in VS Code.

---

#### `amp.ui.zoomLevel`

**Type:** Number  
**Default:** `1`  
**Source:** Manual  
**Scope:** Editor only

**Description:** Zoom level for Amp UI in VS Code.

---

#### `amp.debugLogs`

**Type:** Boolean  
**Default:** `false`  
**Source:** Manual  
**Scope:** Editor only

**Description:** Enable debug logging in VS Code extension.

---

## Undocumented Settings

These settings are present in the CLI code but not officially documented. They may be experimental, internal, or hidden features.

### Core Settings

#### `amp.url`

**Type:** String (URL)  
**Default:** `"https://ampcode.com"`  
**Source:** main.js:5653 (CQ6 registry)  
**Visibility:** Hidden (`visible: false`)

**Description:** Base URL for Amp API server. All API requests (except OpenRouter) are routed through this server.

**Example:**
```json
{
  "amp.url": "https://custom-amp-server.company.com"
}
```

**Notes:**
- Also configurable via `AMP_URL` environment variable
- Set during login flow and persisted
- Critical for proxy configuration
- See [endpoints.md](./endpoints.md) for endpoint details

---

### Anthropic Model Configuration

#### `amp.anthropic.interleavedThinking.enabled`

**Type:** Boolean  
**Default:** `false`  
**Source:** main.js:5653 (CQ6 registry)  
**Visibility:** Hidden

**Description:** Enable interleaved thinking mode for Claude 4, allowing thinking between tool calls.

**Example:**
```json
{
  "amp.anthropic.interleavedThinking.enabled": true
}
```

**Notes:**
- Experimental feature for Claude 4
- Allows model to "think" between tool executions
- May improve reasoning quality but increase latency

---

#### `amp.anthropic.temperature`

**Type:** Number  
**Default:** `1`  
**Source:** main.js:5653 (registry), main.js:1225 (runtime check)  
**Visibility:** Hidden

**Description:** Temperature setting for Claude when thinking mode is disabled.

**Example:**
```json
{
  "amp.anthropic.thinking.enabled": false,
  "amp.anthropic.temperature": 0.7
}
```

**Notes:**
- **Only applies when `amp.anthropic.thinking.enabled` is explicitly set to `false`**
- When thinking is enabled (the default), this setting is ignored
- Range typically 0.0 to 1.0
- Lower values = more deterministic, higher = more creative
- Thinking mode has its own internal temperature/sampling strategy

---

### Model & Agent Configuration

#### `amp.experimental.agentMode`

**Type:** String  
**Default:** Unset (uses CLI default mode)  
**Source:** main.js:6422-6435, 1294-1308  
**Visibility:** Experimental

**Description:** Override agent mode for the thread/session. Influences system prompt, toolset, and model selection.

**Known Modes:**

**Production Modes:**
- `smart` - Balanced performance and capability
- `fast` - Optimized for speed
- `free` - Free tier models

**Experimental Modes:**
- `geppetto` - Experimental mode
- `pinocchio` - Experimental mode
- `claudius` - Experimental mode
- `opus4.1` - Claude Opus 4.1 early access
- `bolt` - Experimental fast mode
- `hydrogen` - Experimental mode
- `kashmir` - Experimental mode
- `gossip` - Experimental mode
- `sonomoon` - Experimental mode
- `gronk-fast` - Groq-based fast mode

**Mode Variants:**
- `:main` - Main agent variant
- `:search` - Search-optimized variant
- `:main+search` - Combined variant

**Example:**
```json
{
  "amp.experimental.agentMode": "fast"
}
```

**Notes:**
- Experimental modes may change or be removed
- Not all modes may be available in all CLI versions
- Some modes may require specific API access

---

### Tools & Execution

#### `amp.tools.inactivityTimeout`

**Type:** Number (seconds)  
**Default:** `300`  
**Source:** main.js:5653 (CQ6 registry)  
**Visibility:** Hidden

⚠️ **VERIFICATION WARNING:** Present in registry but no runtime usage found. May be related to `amp.tools.stopTimeout` from manual. Needs manual testing to confirm functionality.

**Description:** Timeout for tool inactivity. Internal throttling/timer mechanism.

**Example:**
```json
{
  "amp.tools.inactivityTimeout": 600
}
```

**Notes:**
- May be related to or same as documented `amp.tools.stopTimeout`
- Controls how long to wait before timing out inactive tool execution
- **Verification status:** Unverified - needs manual testing

---

### Terminal & Input

#### `amp.dangerouslyAllowAll`

**Type:** Boolean  
**Default:** `false`  
**Source:** main.js:1244-1290 (permissions map)  

**Description:** Bypass all permission prompts and execute all tools without asking.

**Example:**
```json
{
  "amp.dangerouslyAllowAll": true
}
```

**⚠️ WARNING:**
- Extremely dangerous - allows unrestricted tool execution
- Bypasses all security controls
- Only use in fully trusted, sandboxed environments
- Also available as CLI flag

---

### Experimental Features

#### `amp.experimental.cli.nativeSecretsStorage.enabled`

**Type:** Boolean  
**Default:** `false`  
**Source:** main.js:5653 (qy1), 5650-5653  
**Visibility:** Experimental

**Description:** Use OS native keyring for secrets storage instead of file-based storage.

**Example:**
```json
{
  "amp.experimental.cli.nativeSecretsStorage.enabled": true
}
```

**Notes:**
- Uses `@napi-rs/keyring` for secure OS-level secret storage
- When disabled, falls back to file-based secrets
- Supported platforms: macOS (Keychain), Windows (Credential Manager), Linux (libsecret)

---

### Debugging & Development

#### `amp.debug.httpLogging`

**Type:** Boolean  
**Default:** `false`  
**Source:** main.js:6434-6440 (FZ)  
**Visibility:** Internal

**Description:** Enable HTTP wire logging for debugging network requests.

**Example:**
```json
{
  "amp.debug.httpLogging": true
}
```

**Notes:**
- Logs all HTTP requests and responses
- Useful for debugging API issues
- May expose sensitive data in logs (API keys, request bodies)

---

### Integration Settings

#### `amp.openrouter.apiKey`

**Type:** String  
**Source:** Code references to OpenRouter configuration  
**Visibility:** Undocumented

**Description:** API key for OpenRouter direct connection (not proxied through amp.url).

**Example:**
```json
{
  "amp.openrouter.apiKey": "sk-or-xxxxx"
}
```

**Notes:**
- Required to use OpenRouter models (Kimi K2, Sonoma Sky)
- Also configurable via `OPENROUTER_API_KEY` environment variable
- This is NOT the same as `AMP_API_KEY`
- See [endpoints.md](./endpoints.md) for OpenRouter details

---

#### `amp.jetbrains.skipInstall`

**Type:** Boolean  
**Default:** `false`  
**Source:** main.js:6405-6407  
**Visibility:** Internal

**Description:** Skip JetBrains plugin installation prompts.

**Example:**
```json
{
  "amp.jetbrains.skipInstall": true
}
```

**Notes:**
- Suppresses JetBrains IDE plugin install flow
- Set via installer UI "Always Skip" option

---

### Internal Settings

#### `amp.internal.scaffoldCustomizationFile`

**Type:** String (file path)  
**Default:** Unset  
**Source:** main.js:860-866, 1297-1311  
**Visibility:** Internal

**Description:** Path to YAML file for system prompt scaffold customization.

**Example:**
```json
{
  "amp.internal.scaffoldCustomizationFile": "/path/to/custom-scaffold.yaml"
}
```

**Notes:**
- CLI will create/populate file on first use
- Advanced feature for customizing agent behavior
- YAML format expected

---

## Environment Variables

Environment variables can configure the CLI without modifying the settings file. They take precedence over file-based settings.

### Documented Environment Variables

These are documented in CLI help output (`amp --help`) and official documentation.

#### `AMP_API_KEY`

**Type:** String  
**Source:** main.js:5678-5685 (help), 6426-6443 (login/logout)  

**Description:** API key for Amp authentication.

**Example:**
```bash
export AMP_API_KEY=sk-amp-xxxxx
```

**Notes:**
- Required for all API requests
- Obtained via `amp login` command
- Stored securely after login (or in file if native storage disabled)

---

#### `AMP_URL`

**Type:** String (URL)  
**Default:** `https://ampcode.com`  
**Source:** main.js:5678-5685, 6441-6447  

**Description:** Override Amp server URL.

**Example:**
```bash
export AMP_URL=https://custom-amp-server.company.com
```

**Notes:**
- Overrides `amp.url` setting
- Set during login and persisted to settings
- See [endpoints.md](./endpoints.md) for endpoint details

---

#### `AMP_LOG_LEVEL`

**Type:** String enum: `"error"` | `"warn"` | `"info"` | `"debug"` | `"trace"`  
**Default:** `"info"`  
**Source:** main.js:5678-5685, 5600-5610 (logging init)  

**Description:** Set CLI log level.

**Example:**
```bash
export AMP_LOG_LEVEL=debug
```

---

#### `AMP_LOG_FILE`

**Type:** String (file path)  
**Default:** `~/.amp/amp.log`  
**Source:** main.js:5678-5685  

**Description:** Path to CLI log file.

**Example:**
```bash
export AMP_LOG_FILE=/var/log/amp/cli.log
```

---

#### `AMP_SETTINGS_FILE`

**Type:** String (file path)  
**Default:** `~/.amp/settings.json`  
**Source:** main.js:5678-5685, 5631-5653  

**Description:** Path to settings JSON/JSONC file.

**Example:**
```bash
export AMP_SETTINGS_FILE=/etc/amp/settings.json
```

---

#### Standard Node.js Variables

**`HTTP_PROXY` / `HTTPS_PROXY`**

**Type:** String (URL)  
**Description:** HTTP/HTTPS proxy for network requests.

**Example:**
```bash
export HTTPS_PROXY=http://proxy.company.com:8080
```

---

**`NODE_EXTRA_CA_CERTS`**

**Type:** String (file path)  
**Description:** Additional CA certificates for SSL/TLS validation.

**Example:**
```bash
export NODE_EXTRA_CA_CERTS=/etc/ssl/certs/corporate-ca.pem
```

**Notes:**
- Required for corporate proxies with custom SSL certificates

---

### Undocumented Environment Variables

These are found in the code but not officially documented.

#### `AMP_CLI_STDOUT_DEBUG`

**Type:** Boolean (any value enables)  
**Source:** main.js:5600-5610 (logger setup)  

**Description:** Emit logs to console (stdout) in addition to log file.

**Example:**
```bash
export AMP_CLI_STDOUT_DEBUG=1
amp "Test message"  # Logs appear in terminal
```

---

#### `AMP_DEBUG`

**Type:** Boolean (any value enables)  
**Source:** main.js:5601 (error handling path)  

**Description:** Enable extra debugging features and error detail.

**Example:**
```bash
export AMP_DEBUG=1
```

---

#### `AMP_SKIP_UPDATE_CHECK`

**Type:** Boolean (any value enables)  
**Source:** main.js:5757-5765 (update service), 6449-6456  

**Description:** Disable automatic update checks.

**Example:**
```bash
export AMP_SKIP_UPDATE_CHECK=1
```

**Notes:**
- Overrides `amp.updates.mode` setting
- Useful for air-gapped or controlled environments

---

#### `AMP_HOME`

**Type:** String (directory path)  
**Default:** `~/.amp`  
**Source:** main.js:5726-5746, 5748-5757  

**Description:** Override Amp installation/bootstrap directory.

**Example:**
```bash
export AMP_HOME=/opt/amp
```

**Notes:**
- Changes where CLI stores data, settings, and logs
- Affects installation paths

---

#### `AMP_VERSION`

**Type:** String (version number)  
**Source:** main.js:5751-5757 (W79)  

**Description:** Target version for bootstrap/update.

**Example:**
```bash
export AMP_VERSION=0.0.1760000000
```

**Notes:**
- Used during update process
- Pins to specific version
- Development/testing feature

---

#### `AMP_TEST_UPDATE_STATUS`

**Type:** String  
**Source:** main.js:5750-5760  

**Description:** Force specific update status for testing.

**Example:**
```bash
export AMP_TEST_UPDATE_STATUS=available
```

**Notes:**
- Development/testing only
- Forces update UI state

---

#### `OPENROUTER_API_KEY`

**Type:** String  
**Source:** OpenRouter configuration references  

**Description:** API key for OpenRouter (alternative to setting).

**Example:**
```bash
export OPENROUTER_API_KEY=sk-or-xxxxx
```

**Notes:**
- Alternative to `amp.openrouter.apiKey` setting
- Required for OpenRouter models
- See [endpoints.md](./endpoints.md) for OpenRouter details

---

#### `AMP_TOOLBOX` / `TOOLBOX_ACTION`

**Type:** String  
**Source:** Code references to JetBrains integration  

**Description:** JetBrains Toolbox integration environment variables.

**Notes:**
- Used for JetBrains IDE integration protocol
- Internal integration mechanism

---

#### `AMP_PWD`

**Type:** String (directory path)  
**Source:** main.js:6421-6422  

**Description:** Override working directory on CLI launch.

**Example:**
```bash
export AMP_PWD=/path/to/project
amp "List files"
```

**Notes:**
- CLI changes to this directory on startup
- Affects relative file paths in commands

---

## Configuration Examples

### Minimal Configuration

```json
{
  "amp.notifications.enabled": true,
  "amp.git.commit.coauthor.enabled": true
}
```

---

### Advanced Configuration

```json
{
  "amp.url": "https://ampcode.com",
  "amp.notifications.enabled": true,
  "amp.notifications.system.enabled": true,
  "amp.anthropic.thinking.enabled": true,
  "amp.anthropic.interleavedThinking.enabled": false,
  "amp.git.commit.ampThread.enabled": true,
  "amp.git.commit.coauthor.enabled": true,
  "amp.todos.enabled": true,
  "amp.tools.disable": ["browser_navigate"],
  "amp.tools.stopTimeout": 600,
  "amp.terminal.commands.nodeSpawn.loadProfile": "daily",
  "amp.updates.mode": "warn",
  "amp.permissions": [
    {
      "tool": "Bash",
      "action": "ask",
      "reason": "Confirm shell commands for security"
    },
    {
      "tool": "edit_file",
      "action": "allow"
    }
  ],
  "amp.mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "@modelcontextprotocol/server-filesystem",
        "/home/user/projects"
      ]
    }
  }
}
```

---

### OpenRouter Model Usage

```json
{
  "amp.openrouter.apiKey": "sk-or-xxxxx"
}
```

Or via environment:
```bash
export OPENROUTER_API_KEY=sk-or-xxxxx
amp --model kimi/kimi-k2 "Analyze this code"
```

---

### Native Secrets Storage

```json
{
  "amp.experimental.cli.nativeSecretsStorage.enabled": true
}
```

**Benefits:**
- More secure than file-based storage
- Integrates with OS keychain/credential manager
- Encrypted at rest by OS

**Supported Platforms:**
- macOS: Keychain
- Windows: Credential Manager
- Linux: libsecret (requires D-Bus secret service)

---

### Permission Configuration

```json
{
  "amp.permissions": [
    {
      "tool": "all",
      "action": "ask",
      "reason": "Default: ask for all tools"
    },
    {
      "tool": "Read",
      "action": "allow",
      "reason": "Reading files is safe"
    },
    {
      "tool": "Bash",
      "action": "ask",
      "reason": "Always confirm shell commands"
    },
    {
      "tool": "edit_file",
      "action": "allow",
      "reason": "File edits are reversible via git"
    }
  ]
}
```

**Permission Actions:**
- `allow` - Execute without prompting
- `ask` - Prompt user for approval
- `reject` - Never allow execution
- `delegate` - Delegate to another agent/system

---

### MCP Server Setup

```json
{
  "amp.mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "@modelcontextprotocol/server-filesystem",
        "/home/user/projects",
        "/home/user/documents"
      ]
    },
    "database": {
      "command": "node",
      "args": ["/usr/local/bin/mcp-postgres-server.js"],
      "env": {
        "DB_HOST": "localhost",
        "DB_PORT": "5432",
        "DB_NAME": "myapp",
        "DB_USER": "user",
        "DB_PASSWORD": "secret"
      }
    },
    "custom-api": {
      "command": "python3",
      "args": ["/path/to/custom-mcp-server.py"],
      "env": {
        "API_KEY": "xxxxx"
      }
    }
  }
}
```

**Notes:**
- Each server runs as a separate child process
- Servers communicate via stdio (MCP protocol)
- Environment variables can pass configuration to servers
- Servers expose additional tools to the agent

---

### Debug Configuration

```json
{
  "amp.debug.httpLogging": true
}
```

Plus environment:
```bash
export AMP_LOG_LEVEL=debug
export AMP_CLI_STDOUT_DEBUG=1
export AMP_DEBUG=1

amp "Test request"
```

**Output:**
- Detailed HTTP request/response logs
- Debug-level application logs
- Logs appear in terminal and log file

---

## Notes on Settings

### Visibility Flags

Settings in the code registry have visibility flags:

- **Visible** (`visible: true`) - Intended for user configuration
- **Hidden** (`visible: false`) - Internal, but functional if set
- **Experimental** - May change or be removed in future versions
- **Deprecated** - Still functional but marked for removal

### Setting Scopes

Settings apply to different contexts:

- **CLI Only** - Only used by command-line interface
- **Editor Only** - Only used by VS Code extension
- **Both** - Used by both CLI and editor

**Editor-only settings** (not applicable to CLI):
- `amp.tab.enabled`
- `amp.ui.zoomLevel`
- `amp.debugLogs`

### Migration Warnings

**No deprecated settings with migration paths** were found in this version. The CLI appears to maintain backward compatibility without explicit deprecation warnings.

CLI flags (not settings) may have deprecation notices:
- `--format` flag (deprecated)
- `--interactive` flag (deprecated)

### Precedence Order

Configuration is loaded with the following precedence (highest to lowest):

1. **Environment variables** (e.g., `AMP_URL`, `AMP_API_KEY`)
2. **CLI flags** (e.g., `--model`, `--no-stream`)
3. **Settings file** (`~/.amp/settings.json` or `AMP_SETTINGS_FILE`)
4. **Default values** (from code registry)

### Settings Validation

The CLI validates settings on load:
- Type checking (boolean, string, number, object, array)
- Enum validation (for settings with fixed values)
- Path validation (for file/directory settings)

Invalid settings will:
- Log a warning
- Fall back to default value
- Not block CLI execution (unless critical like API key)

---

## Verification Status

This documentation was created through automated extraction and Oracle-assisted analysis. However, **not all settings could be verified** to actually work in the runtime code.

### Verification Categories

- ✅ **VERIFIED** - Confirmed to work via runtime code analysis (73% of settings)
- ⚠️ **UNVERIFIED** - In registry but runtime usage not found (16% of settings)
- 🔴 **DISCREPANCY** - Registry default ≠ runtime default (4% of settings)
- ❌ **HARDCODED** - Setting may be ignored by runtime (4% of settings)

### Critical Issues Found

1. **`amp.anthropic.thinking.enabled`** 🔴
   - Registry default: `false`
   - **Actual runtime default: `true`**
   - Impact: Thinking is ON by default, not off
   - Resolution: Manual is correct, registry is misleading

2. **`amp.todos.enabled`** ❌
   - Setting exists but runtime may hardcode `true`
   - Impact: Setting may not disable todos
   - Status: Needs manual testing

### Unverified Settings (Require Testing)

These settings exist but couldn't be verified in runtime code:
- `amp.notifications.enabled` (may be overridden by CLI flag)
- `amp.tools.disable`
- `amp.tools.stopTimeout` / `amp.tools.inactivityTimeout`
- `amp.debug.httpLogging`

### Detailed Verification Report

For complete verification details including:
- Line numbers for all runtime usage
- Code snippets showing how settings are read
- Testing methodology for unverified settings
- Known issues and workarounds

**See:** [Settings Verification Status](./settings-verification-status.md)

### Recommendations

1. **Use with confidence (Verified):**
   - All documented settings except those marked with warnings
   - All core settings (`amp.url`, `amp.permissions`, etc.)
   - Git integration settings
   - Most environment variables

2. **Test before production (Unverified):**
   - Any setting marked with ⚠️ warning in this document
   - Settings not explicitly verified in the verification status doc

3. **Report issues:**
   - If a setting doesn't work as documented, check verification status
   - File issue with Amp team if verified setting doesn't work
   - Contribute testing results for unverified settings

---

## Related Resources

- [Amp Manual - Configuration](https://ampcode.com/manual#configuration) - Official documentation
- [Settings Verification Status](./settings-verification-status.md) - Detailed verification report
- [API Endpoints Reference](./endpoints.md) - HTTP endpoint documentation
- [MCP Specification](https://modelcontextprotocol.io) - Model Context Protocol
- [Agent Modes Documentation](https://ampcode.com/manual#agent-modes) - Agent mode details

---

## Appendix: Complete Settings Registry (CQ6)

The settings registry object (CQ6) at main.js:5653 contains all settings with their metadata. Below is a summary of all registry entries:

| Setting Key | Type | Default | Visible | Source Line |
|-------------|------|---------|---------|-------------|
| `url` | string | `"https://ampcode.com"` | false | 5653 |
| `anthropic.thinking.enabled` | boolean | `true` (runtime) | false | 5653, 1225 |
| `anthropic.interleavedThinking.enabled` | boolean | `false` | false | 5653 |
| `anthropic.temperature` | number | `1` | false | 5653 |
| `notifications.enabled` | boolean | `true` | true | 5653 |
| `notifications.system.enabled` | boolean | `true` | true | 5653 |
| `todos.enabled` | boolean | `true` | false | 5653 |
| `mcpServers` | object | (example) | true | 5653 |
| `tools.disable` | array | `["browser_navigate", "builtin:edit_file"]` | true | 5653 |
| `tools.inactivityTimeout` | number | `300` | false | 5653 |

**Notes:**
- `Visible: false` = Hidden setting (not shown in UI/docs)
- `Visible: true` = Public setting (intended for users)
- Other settings are loaded/used outside the registry
- **Important:** Registry defaults may differ from runtime defaults. For example, `anthropic.thinking.enabled` has a registry default of `false` but a runtime default of `true` (via `?? true` fallback when unset)

---

**Generation Details:**
- CLI Version: 0.0.1761153678-gfa55cf
- Bundle: `node_modules/@sourcegraph/amp/dist/main.js`
- Analysis Method: Oracle-assisted extraction with manual cross-reference
- Manual URL: https://ampcode.com/manual#configuration
- Verification: Settings verified against code (main.js:5653, CQ6) and manual
- Last Updated: 2025-01-21

---

## Version History

### v0.0.1761153678-gfa55cf (2025-01-21)
- Initial extraction
- All documented and undocumented settings catalogued
- Environment variables comprehensively documented
- Cross-referenced with Amp Manual
- Oracle-verified source line numbers
