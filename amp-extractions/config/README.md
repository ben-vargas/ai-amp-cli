# AMP CLI Configuration Documentation

**Version:** 0.0.1761153678-gfa55cf  
**Last Updated:** 2025-01-21  
**Source:** Automated extraction + Oracle-assisted analysis

---

## Overview

This directory contains comprehensive documentation of ALL configuration options for the AMP CLI, including both documented and undocumented settings, API endpoints, and verification status.

---

## Documents

### 📘 [endpoints.md](./endpoints.md)
**Complete API endpoint reference**

Documents all HTTP endpoints used by the AMP CLI:
- Model inference endpoints (7 providers + OpenRouter)
- User & thread management APIs
- GitHub integration proxies
- Real-time SSE connections
- Request headers and proxy configuration

**Use this for:**
- Configuring network proxies
- Understanding data flow
- Debugging connection issues
- Setting up firewall rules

**Status:** ✅ Fully verified via code analysis

---

### ⚙️ [settings.md](./settings.md)
**Complete settings reference**

Documents ALL configuration settings (45 total):
- Documented settings (17) - Official from Amp Manual
- Undocumented settings (11) - Found in code registry
- Environment variables (17) - Both documented and hidden

**Use this for:**
- Configuring the CLI
- Finding advanced options
- Understanding setting behavior
- Customizing agent modes

**Status:** ⚠️ 73% verified (see verification status below)

---

### ✓ [settings-verification-status.md](./settings-verification-status.md)
**Detailed verification report**

Tracks which settings have been verified to actually work:
- ✅ VERIFIED (73%) - Confirmed via runtime code
- ⚠️ UNVERIFIED (16%) - In registry but usage not found
- 🔴 DISCREPANCY (4%) - Registry ≠ runtime defaults
- ❌ HARDCODED (4%) - Setting may be ignored

**Use this for:**
- Checking if a setting is reliable
- Understanding verification methodology
- Finding known issues
- Prioritizing manual testing

**Critical findings:**
- `amp.anthropic.thinking.enabled` defaults to `true`, not `false`
- `amp.todos.enabled` may be hardcoded (setting ignored)

---

### 🧪 [testing-guide.md](./testing-guide.md)
**Manual testing procedures**

Step-by-step guides for testing unverified settings:
- General testing methodology
- Specific test procedures (6 settings)
- Result reporting templates
- Safety guidelines

**Use this for:**
- Testing unverified settings
- Contributing verification results
- Confirming settings work
- Debugging setting issues

**Priority tests:**
1. `amp.todos.enabled` (high - may be broken)
2. `amp.notifications.enabled` (medium)
3. `amp.tools.disable` (medium)

---

### 📋 [01-prompt-extract-config.md](./01-prompt-extract-config.md)
**Extraction template**

Original prompt used to extract this documentation:
- Detailed requirements
- Oracle verification steps
- Success criteria
- Maintenance procedures

**Use this for:**
- Updating documentation for new CLI versions
- Understanding extraction methodology
- Reproducing analysis
- Creating similar extractions

---

## Quick Start

### Find a Setting

1. **Check [settings.md](./settings.md)** - Find setting description and default
2. **Check [settings-verification-status.md](./settings-verification-status.md)** - See if it's verified to work
3. **If unverified:** Use [testing-guide.md](./testing-guide.md) to test it manually

### Configure Network Proxy

1. **Read [endpoints.md](./endpoints.md)** - Understand what endpoints are used
2. **Configure proxy** - Route `amp.url/*` traffic (except OpenRouter)
3. **Test connection** - Run `amp login` and test inference

### Debug Setting Issue

1. **Enable debug logging:**
   ```bash
   export AMP_LOG_LEVEL=debug
   export AMP_CLI_STDOUT_DEBUG=1
   ```

2. **Check if setting is read:**
   ```bash
   amp "test prompt" 2>&1 | grep "your.setting.key"
   ```

3. **Consult verification status** - Is setting verified to work?

4. **Follow testing guide** - Test setting manually if unverified

---

## Known Issues

### Critical Issues

#### 1. `amp.anthropic.thinking.enabled` Default Mismatch
**Problem:** Registry shows `false`, but runtime defaults to `true`  
**Impact:** Thinking is ON by default, contrary to registry  
**Status:** 🔴 DISCREPANCY  
**Resolution:** Explicitly set to `false` if you want thinking disabled  
**Details:** [settings-verification-status.md#amp-anthropic-thinking-enabled](./settings-verification-status.md#amp-anthropic-thinking-enabled)

#### 2. `amp.todos.enabled` May Be Non-Functional
**Problem:** Runtime appears to hardcode `true`, ignoring setting  
**Impact:** Cannot disable todos via setting  
**Status:** ❌ HARDCODED (needs testing)  
**Resolution:** Manual testing required to confirm  
**Details:** [settings-verification-status.md#amp-todos-enabled](./settings-verification-status.md#amp-todos-enabled)

### Unverified Settings (Needs Testing)

These settings exist but couldn't be verified:
- `amp.notifications.enabled` - May be overridden by CLI flag
- `amp.tools.disable` - Exists in registry, usage unclear
- `amp.tools.stopTimeout` / `amp.tools.inactivityTimeout` - Relationship unclear
- `amp.debug.httpLogging` - Mentioned but usage not found

**Help needed:** Test these settings using [testing-guide.md](./testing-guide.md)

---

## Statistics

### Documentation Coverage

**Endpoints:**
- 8 model providers documented (including OpenRouter)
- 6 internal API categories
- 100% of endpoints verified ✅

**Settings:**
- 45 total settings documented
- 33 verified (73%) ✅
- 7 unverified (16%) ⚠️
- 2 with issues (4%) 🔴❌
- 3 editor-only (7%) 📝

**Environment Variables:**
- 17 total variables documented
- 14 verified (82%) ✅
- 3 unverified (18%) ⚠️

### Verification Confidence

- **High Confidence (VERIFIED):** 73%
- **Medium Confidence (UNVERIFIED):** 16%
- **Issues Found:** 4%
- **Not Applicable (Editor):** 7%

---

## Maintenance

### Updating for New Versions

When a new AMP CLI version is released:

1. **Update npm-packages:** Install new version in `npm-packages/` directory

2. **Re-run extraction:** Use [01-prompt-extract-config.md](./01-prompt-extract-config.md) as template

3. **Update documents:**
   - [endpoints.md](./endpoints.md) - Check for new providers/endpoints
   - [settings.md](./settings.md) - Check for new/removed settings
   - [settings-verification-status.md](./settings-verification-status.md) - Re-verify all settings

4. **Update line numbers:** Code locations change with each version

5. **Re-test unverified settings:** Some may become verified in new versions

6. **Check for deprecations:** Settings may be removed or replaced

### Contributing

Help improve this documentation:

1. **Test unverified settings** - Follow [testing-guide.md](./testing-guide.md)
2. **Report results** - Update [settings-verification-status.md](./settings-verification-status.md)
3. **File issues** - Report bugs if settings don't work
4. **Submit corrections** - Fix errors or add missing info

---

## Methodology

### How This Was Created

1. **Oracle-assisted extraction:**
   - Used Oracle AI to analyze minified JavaScript bundle
   - Searched for endpoint URLs, settings registry, runtime usage
   - Cross-referenced with official manual

2. **Line number verification:**
   - Every finding includes source line number
   - Code snippets show actual usage
   - Verified via grep and manual inspection

3. **Manual cross-reference:**
   - Compared extracted data with Amp Manual
   - Identified discrepancies
   - Documented verification confidence

4. **Testing methodology:**
   - Created systematic testing procedures
   - Prioritized unverified settings
   - Designed result reporting templates

### Accuracy Notes

**What we can verify:**
- Settings present in registry ✅
- Runtime code that reads settings ✅
- Default values in code ✅
- Endpoint URLs used ✅

**What we cannot verify (without testing):**
- Whether settings actually change behavior ⚠️
- Edge cases and interactions ⚠️
- Settings with no obvious runtime reads ⚠️

**Why minified code is hard:**
- Variable names obfuscated (e.g., `CQ6`, `!0`)
- No comments or documentation
- Some logic may be inferred, not explicit

---

## Related Resources

### Official Documentation
- [Amp Manual](https://ampcode.com/manual) - Official user guide
- [Amp Manual - Configuration](https://ampcode.com/manual#configuration) - Settings reference
- [Model Context Protocol](https://modelcontextprotocol.io) - MCP specification

### This Repository
- [Endpoints Reference](./endpoints.md)
- [Settings Reference](./settings.md)
- [Verification Status](./settings-verification-status.md)
- [Testing Guide](./testing-guide.md)
- [Extraction Prompt](./01-prompt-extract-config.md)

---

## Support

### For Setting Issues

1. **Check verification status** - Is setting verified?
2. **Enable debug logging** - Look for setting being read
3. **Test manually** - Follow testing guide
4. **Check manual** - Compare with official docs
5. **File issue** - Report to Amp team if bug

### For Network Issues

1. **Check endpoints.md** - Confirm endpoint paths
2. **Test direct connection** - Bypass proxy
3. **Check headers** - Ensure auth headers present
4. **Verify SSL/TLS** - Corporate proxies may need custom CA
5. **Test OpenRouter separately** - It's not proxied through amp.url

### For Documentation Errors

1. **Verify against code** - Check line numbers
2. **Test setting manually** - Confirm behavior
3. **Report discrepancy** - File issue or PR
4. **Update verification status** - Add test results

---

## License & Attribution

**Generated from:** AMP CLI v0.0.1761153678-gfa55cf  
**Source code:** `node_modules/@sourcegraph/amp/dist/main.js`  
**Amp CLI:** © Sourcegraph  
**Documentation:** Community-contributed extraction

This documentation is a community effort to understand and document the AMP CLI configuration. It is not officially maintained by Sourcegraph and may contain errors or become outdated.

**Accuracy disclaimer:** Settings marked as UNVERIFIED have not been tested. Use verification status to understand confidence level.

---

## Version History

### v0.0.1761153678-gfa55cf (2025-01-21)
- Initial comprehensive extraction
- 4 documentation files created
- 45 settings documented
- 73% verification rate achieved
- 2 critical issues identified
- Testing guide created

---

**Last Updated:** 2025-01-21  
**Maintainer:** Community  
**Verification Level:** 73% verified, 16% unverified, 4% with issues
