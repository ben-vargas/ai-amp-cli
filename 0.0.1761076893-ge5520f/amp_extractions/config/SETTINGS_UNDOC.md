# Undocumented Amp Settings

This document lists **undocumented and experimental** settings for Amp CLI that are not listed in the [official manual](https://ampcode.com/manual).

> ⚠️ **Warning**: These settings are internal, experimental, or deprecated. They may change or be removed without notice. Use at your own risk.

---

## General Settings

### `amp.url`
- **Type**: `string`
- **Default**: `"https://ampcode.com"`
- **Visibility**: Hidden (internal)
- **Description**: The Amp server URL to connect to. This is the base URL for the Amp service API.

---

## Anthropic Provider Settings

### `amp.anthropic.thinking.enabled`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden (internal)
- **Description**: Enable Claude thinking process output for debugging. When enabled, you'll see the model's internal reasoning process.

### `amp.anthropic.interleavedThinking.enabled`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden (internal)
- **Description**: Enable interleaved thinking for Claude 4 models. This allows reasoning between tool calls rather than only at the beginning.

### `amp.anthropic.temperature`
- **Type**: `number`
- **Default**: `1.0`
- **Visibility**: Hidden (internal)
- **Description**: Temperature setting for Anthropic models (0.0 = deterministic, 1.0 = creative). **Note**: Only takes effect when thinking is disabled. Internal use only.

### `amp.anthropic.provider`
- **Type**: `string`
- **Default**: `"anthropic"`
- **Visibility**: Hidden (internal)
- **Description**: Which provider to use for Anthropic Claude inference. Options: `"anthropic"` or `"vertex"` (for Google Cloud Vertex AI).

---

## Model Configuration (Experimental/Internal)

### `amp.internal.model`
- **Type**: `string`
- **Default**: `undefined`
- **Visibility**: Hidden (internal)
- **Description**: Override the primary model used for inference. Format: `"provider:model"` (e.g., `"anthropic:claude-sonnet-4-5-20250929"`). Parsed by splitting on `:` to extract provider and model name.

### `amp.internal.oracleModel`
- **Type**: `string`
- **Default**: `"gpt-5"`
- **Visibility**: Hidden (internal)
- **Description**: Override which model the Oracle tool uses. Defaults to GPT-5. Can be set to any OpenAI model that supports the Responses API (e.g., `"gpt-5-mini"`, `"gpt-5-nano"`, `"o3"`, `"o3-mini"`).

### `amp.internal.primaryModel`
- **Type**: `string`
- **Default**: `undefined`
- **Visibility**: Hidden (internal - DEPRECATED)
- **Description**: **[DEPRECATED]** This setting has been migrated to `amp.experimental.agentMode`. The CLI will automatically migrate this on startup if set to known values like `"anthropic/claude-opus-4-1-20250805"` → `"opus4.1"` or `"openai/gpt-5"` → `"geppetto:main"`.

### `amp.internal.visibleModes`
- **Type**: `array`
- **Default**: `[]`
- **Visibility**: Hidden (internal)
- **Description**: Array of agent mode keys to make visible in the UI mode selector, beyond the default visible modes. Can include experimental modes like `"fast"`, `"gronk-fast:main"`, `"bolt:main"`, `"geppetto:main"`, `"pinocchio:main"`, `"claudius:main"`, `"opus4.1"`, `"hydrogen:main"`, `"kashmir:main"`, `"gossip:main"`, `"sonomoon:main"`, etc.

### `amp.internal.showCost`
- **Type**: `boolean`
- **Default**: `true`
- **Visibility**: Hidden (internal)
- **Description**: Whether to display cost/credit information in the UI. Can be toggled with keyboard shortcut.

---

## Experimental Agent Modes

### `amp.experimental.agentMode`
- **Type**: `string`
- **Default**: `undefined`
- **Visibility**: Hidden (internal)
- **Description**: Set experimental agent mode configuration. Overrides the default mode selection. Available modes include:
  - **Production modes**: `"smart"`, `"fast"`, `"free"`
  - **Experimental modes**: 
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
    - `"kashmir:main"` - Kimi K2 Instruct
    - `"gossip:main"` - GPT OSS 120B
    - `"sonomoon:main"` - Sonoma Sky Alpha
    - Plus `:search` and `:main+search` variants

---

## Terminal & Input Settings

### `amp.submitOnEnter`
- **Type**: `boolean`
- **Default**: `true`
- **Visibility**: Hidden (internal)
- **Description**: Whether to submit messages on Enter key (true) or require Ctrl+Enter (false).

### `amp.terminal.commands.nodeSpawn.detachedLogsDirectory`
- **Type**: `string`
- **Default**: `undefined`
- **Visibility**: Hidden (internal)
- **Description**: Directory to store detached command logs when using node-spawn mode for bash commands.

---

## Tab Completion Settings (All Hidden)

### `amp.tab.enabled`
- **Type**: `boolean`
- **Default**: `true`
- **Visibility**: Hidden
- **Description**: Enable tab completion features (inline code suggestions).

### `amp.tab.disabledLanguages`
- **Type**: `array`
- **Default**: `[]`
- **Visibility**: Hidden
- **Description**: List of language IDs for which tab completion is disabled (e.g., `["python", "javascript"]`).

### `amp.tab.disableOnComment`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Disable tab completion when cursor is on a comment line.

### `amp.tab.autoImport.enabled`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Enable automatic import statements feature for tab completion suggestions.

### `amp.tab.experimental.unobtrusiveDiffs`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Experimental: Enable unobtrusive diffs mode for tab suggestions to avoid layout shifts in the editor.

### `amp.tab.experimental.showHotStreakProgress`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Experimental: Show hot streak progress indicator in tab suggestions.

### `amp.tab.experimental.autoAccept.delayMs`
- **Type**: `number`
- **Default**: `0`
- **Visibility**: Hidden
- **Description**: Experimental: Delay in milliseconds before auto-accepting tab suggestions. Set to 0 to disable auto-accept.

### `amp.tab.experimental.disableClientFilters`
- **Type**: `array`
- **Default**: `[]`
- **Visibility**: Hidden
- **Description**: Experimental: Disable specific client-side filters to observe unfiltered model behavior. Used for debugging tab completion filtering logic.

### `amp.tab.diagnosticTracking`
- **Type**: `string`
- **Default**: `"multi_file"`
- **Visibility**: Hidden
- **Description**: Enable diagnostic tracking for tab suggestions. Options: `"disabled"`, `"same_file"`, or `"multi_file"`.

### `amp.tab.hoverDiagnosticSuggestions`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Enable hover diagnostics for Tab completion suggestions.

### `amp.tab.experimental.displayMode`
- **Type**: `string`
- **Default**: `"eager"`
- **Visibility**: Hidden
- **Description**: Control how tab suggestions are displayed. Options:
  - `"eager"` - Show diff immediately
  - `"subtle"` - Show "Tab to Jump" hint first

### `amp.tab.dismissCommandIds`
- **Type**: `array`
- **Default**: `["extension.vim_escape", "vim.escape", "vscode-neovim.escape", "vscode-neovim.escapeKey"]`
- **Visibility**: Hidden
- **Description**: Command IDs to execute when escape key is pressed in modal editors (Vim mode compatibility).

### `amp.tab.verboseLogs`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Enable verbose logging output for tab completion debugging.

---

## Tool Configuration

### `amp.tools.inactivityTimeout`
- **Type**: `number`
- **Default**: `300` (seconds)
- **Visibility**: Hidden
- **Description**: How many seconds of no output to wait before canceling bash commands automatically.

### `amp.todos.enabled`
- **Type**: `boolean`
- **Default**: `true`
- **Visibility**: Hidden
- **Description**: Enable TODO tracking and management features. The agent uses this to plan and track multi-step tasks.

---

## Experimental Features

### `amp.experimental.run_javascript.enabled`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Whether to enable the experimental `run_javascript` tool that allows the agent to execute JavaScript code.

### `amp.experimental.cli.nativeSecretsStorage.enabled`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Use native OS secret storage (keychain/credential manager) instead of the plain-text `secrets.json` file.

### `amp.experimental.reviewTool`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Experimental: Enable the code review tool that performs automated code reviews using a subagent.

### `amp.experimental.tools`
- **Type**: `array`
- **Default**: `[]`
- **Visibility**: Hidden
- **Description**: Enable experimental tools by name. Available experimental tools: `"chat_llm"`.

### `amp.experimental.mcpPermissions`
- **Type**: `unknown`
- **Default**: `undefined`
- **Visibility**: Hidden (DEPRECATED)
- **Description**: **[DEPRECATED]** Automatically migrated to `amp.mcpPermissions` on first run.

---

## Guidance Files

### `amp.guidanceFiles.specs`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: [Experimental] Read `*.spec.md` files adjacent to `AGENT.md` / `AGENTS.md` files and include them in the agent's context.

---

## Hooks System

### `amp.hooks`
- **Type**: `array`
- **Default**: `[]`
- **Visibility**: Hidden
- **Description**: Custom hooks for extending Amp functionality. Advanced feature for programmatic customization.

---

## Deprecated Settings

### `amp.gpt5`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden (DEPRECATED)
- **Description**: **[DEPRECATED]** GPT-5 is no longer available as the main model. It's now the oracle. See https://ampcode.com/news/gpt-5-oracle

---

## JetBrains Integration

### `amp.jetbrains.skipInstall`
- **Type**: `boolean`
- **Default**: `false`
- **Visibility**: Hidden
- **Description**: Skip JetBrains plugin installation flow during startup.

---

## OpenRouter Integration

### `amp.openrouter.apiKey`
- **Type**: `string`
- **Default**: `undefined`
- **Visibility**: Not explicitly listed but referenced in code
- **Description**: API key for OpenRouter provider. Used when running experimental modes that route through OpenRouter.

---

## How to Use These Settings

Edit `~/.config/amp/settings.json` and add settings with the `amp.` prefix:

```json
{
  "amp.git.commit.coauthor.enabled": false,
  "amp.internal.oracleModel": "gpt-5-mini",
  "amp.anthropic.thinking.enabled": true,
  "amp.experimental.agentMode": "geppetto:main",
  "amp.internal.visibleModes": ["fast", "geppetto:main"],
  "amp.tab.enabled": false,
  "amp.experimental.cli.nativeSecretsStorage.enabled": true
}
```

---

## Notes

- Settings marked with `visible: false` in the source code are intentionally hidden from documentation
- Experimental settings (`experimental.*`) are subject to breaking changes
- Internal settings (`internal.*`) are for advanced users and debugging
- Tab completion settings (`tab.*`) are mostly for IDE integration
- Deprecated settings should not be used in new configurations

---

## Related Resources

- [Official Amp Manual](https://ampcode.com/manual)
- [Amp Settings Location](https://ampcode.com/manual#configuration): `~/.config/amp/settings.json`
- [Agent Modes Documentation](https://ampcode.com/news/gpt-5-oracle)

---

**Last Updated**: Generated from Amp CLI v0.0.1760587291-gab1037 source code analysis
