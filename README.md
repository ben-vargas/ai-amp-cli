# Amp CLI Extractions

This repository contains comprehensive documentation extracted from the Amp CLI source code, providing detailed reference material for developers working with or extending the Amp CLI system.

## What is Amp?

Amp is a powerful AI coding agent built by Sourcegraph that helps with software engineering tasks. The Amp CLI provides an intelligent coding assistant powered by various AI models (Claude, GPT-5, Gemini, Grok, and more) with specialized agents for different types of work.

## What's in This Repository?

This repository contains detailed technical documentation extracted from analyzing the Amp CLI source code (`node_modules/@sourcegraph/amp/dist/main.js`). The documentation covers:

- **Configuration system** - All settings, both documented and undocumented
- **API endpoints** - Complete endpoint reference for model inference and services
- **Agent architecture** - System prompts and capabilities of different AI agents
- **Tool definitions** - Detailed specifications of tools available to agents

## Documentation Contents

### Config Settings & Endpoints

- [**amp-extractions/config/settings.md**](amp-extractions/config/settings.md) - Complete reference of all Amp CLI configuration settings, including documented public settings, undocumented internal settings, experimental features, and environment variables
- [**amp-extractions/config/endpoints.md**](amp-extractions/config/endpoints.md) - Comprehensive documentation of all HTTP endpoints used by Amp CLI for model inference, user management, GitHub integration, and internal services

### Agents & Tools

- [**amp-extractions/agents/librarian-system-prompt.md**](amp-extractions/agents/librarian-system-prompt.md) - System prompt for the Librarian agent, a specialized codebase understanding agent for multi-repository analysis
- [**amp-extractions/agents/oracle-system-prompt.md**](amp-extractions/agents/oracle-system-prompt.md) - System prompt for the Oracle agent, an expert advisor using GPT-5 for code reviews, architecture decisions, and complex debugging
- [**amp-extractions/agents/smart-system-prompt.md**](amp-extractions/agents/smart-system-prompt.md) - System prompt for the Smart agent (main/default agent), the primary coding agent that handles most user interactions
- [**amp-extractions/agents/agent-tools.md**](amp-extractions/agents/agent-tools.md) - Complete tool definitions and implementations for Librarian and Oracle agents, including all 7 GitHub tools and specialized capabilities

## Use Cases

This documentation is valuable for:

- **Understanding Amp internals** - Learn how Amp works under the hood
- **Advanced configuration** - Discover experimental and undocumented settings
- **Extending Amp** - Build tools or integrations that work with Amp
- **Debugging** - Troubleshoot issues by understanding the system architecture
- **Research** - Study AI agent design patterns and multi-agent systems

## Repository Structure

```
.
├── README.md
├── amp-extractions/
│   ├── config/
│   │   ├── settings.md      # Complete configuration settings reference
│   │   └── endpoints.md     # API endpoints documentation
│   ├── agents/
│   │   ├── librarian-system-prompt.md  # Librarian agent system prompt
│   │   ├── oracle-system-prompt.md     # Oracle agent system prompt
│   │   ├── smart-system-prompt.md      # Smart agent system prompt (main/default)
│   │   └── agent-tools.md              # Complete tool definitions
│   └── tools/
└── npm-packages/
```

## Source Information

**Generated from**: Amp CLI v0.0.1761076893-ge5520f source code analysis  
**Primary source**: `node_modules/@sourcegraph/amp/dist/main.js` (analyzed artifact, not included in this repository)  
**Verification**: Oracle analysis and manual extraction  
**Last updated**: 2025-01-20

## Related Resources

- [Official Amp Manual](https://ampcode.com/manual)
- [Amp Configuration Guide](https://ampcode.com/manual#configuration)
- [Amp News & Updates](https://ampcode.com/news)

## Community Tools

- [**ai-cli-proxy-api**](https://github.com/ben-vargas/ai-cli-proxy-api) - Proxy solution for using Amp CLI and VS Code extension with ChatGPT Plus/Pro and Claude Pro/Max subscriptions instead of per-token API pricing through Amp's proxy

## Contributing

This is a reference documentation repository. Updates should be made by re-analyzing the Amp CLI source code when new versions are released.

---

**Note**: This repository contains extracted documentation for reference purposes. For official Amp documentation, please refer to [ampcode.com/manual](https://ampcode.com/manual).
