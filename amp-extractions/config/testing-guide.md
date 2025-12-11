# AMP CLI Settings Testing Guide

**Version:** 0.0.1761153678-gfa55cf  
**Last Updated:** 2025-01-21

---

## Purpose

This guide helps you manually test UNVERIFIED settings to determine if they actually work. Results can be contributed back to improve the [Settings Verification Status](./settings-verification-status.md).

---

## Prerequisites

1. **AMP CLI installed** - Version 0.0.1761153678-gfa55cf or compatible
2. **Settings file access** - Know where your `~/.amp/settings.json` is located
3. **Debug logging enabled** - Can enable via environment variables
4. **Backup your config** - Always backup before testing

```bash
# Backup your current settings
cp ~/.amp/settings.json ~/.amp/settings.json.backup
```

---

## General Testing Methodology

For any unverified setting:

### Step 1: Identify Baseline Behavior

Test WITHOUT the setting to establish baseline:

```bash
# Remove setting if present
# Edit ~/.amp/settings.json and remove the test setting

# Test baseline behavior
amp "Your test prompt"
```

Document what happens.

### Step 2: Test Non-Default Value

Add setting with non-default value:

```json
{
  "amp.setting.name": <non-default-value>
}
```

Run same test:

```bash
amp "Your test prompt"
```

### Step 3: Compare Results

Does behavior change?
- **YES** → Setting works! ✅
- **NO** → Setting may not work ⚠️

### Step 4: Enable Debug Logging

Test again with full logging:

```bash
export AMP_LOG_LEVEL=debug
export AMP_CLI_STDOUT_DEBUG=1
export AMP_LOG_FILE=~/.amp/test.log

amp "Your test prompt"

# Check logs for setting key
grep "setting.name" ~/.amp/test.log
```

If setting appears in logs being read → Likely works  
If setting never appears in logs → Likely ignored

---

## Specific Setting Tests

### Test 1: `amp.notifications.enabled`

**Status:** UNVERIFIED  
**Hypothesis:** May be overridden by CLI flag

**Test Setup:**
```json
{
  "amp.notifications.enabled": false
}
```

**Test Steps:**

1. **Baseline** (no setting or setting=true):
   ```bash
   amp "echo hello"
   ```
   Expected: Completion sound plays (if audio enabled on system)

2. **Test** (setting=false):
   ```bash
   amp "echo hello"
   ```
   Expected if works: NO completion sound
   
3. **CLI flag test**:
   ```bash
   amp --notifications=false "echo hello"
   ```
   Does CLI flag work? If yes, compare to setting behavior.

**Results to Document:**
- Does setting disable sounds? (YES/NO)
- Does CLI flag disable sounds? (YES/NO)
- Are they independent or does flag override setting?

---

### Test 2: `amp.tools.disable`

**Status:** UNVERIFIED  
**Hypothesis:** Should disable specific tools

**Test Setup:**
```json
{
  "amp.tools.disable": ["Bash"]
}
```

**Test Steps:**

1. **Baseline** (no setting):
   ```bash
   amp "run 'ls' command"
   ```
   Expected: Agent can use Bash tool, runs `ls`

2. **Test** (Bash disabled):
   ```bash
   amp "run 'ls' command"
   ```
   Expected if works: Agent says Bash tool not available or refuses to run command

3. **Verify tool list**:
   Enable debug logging and check what tools are registered:
   ```bash
   export AMP_LOG_LEVEL=debug
   export AMP_CLI_STDOUT_DEBUG=1
   amp "list your available tools"
   ```
   Check logs for tool registration. Is Bash missing?

**Results to Document:**
- Does setting actually disable tools? (YES/NO)
- Does agent acknowledge tool is unavailable? (YES/NO)
- Do logs show filtered tool list? (YES/NO)

---

### Test 3: `amp.todos.enabled`

**Status:** HARDCODED (possibly non-functional)  
**Hypothesis:** Setting may be ignored, todos always enabled

**Test Setup:**
```json
{
  "amp.todos.enabled": false
}
```

**Test Steps:**

1. **Baseline** (no setting or setting=true):
   ```bash
   amp "create a todo list for implementing user authentication"
   ```
   Expected: Agent can use todo tools

2. **Test** (setting=false):
   ```bash
   amp "create a todo list for implementing user authentication"
   ```
   Expected if works: Agent says todo tool not available
   Expected if broken: Agent still creates todos (setting ignored)

3. **Check system prompt**:
   ```bash
   export AMP_LOG_LEVEL=debug
   export AMP_CLI_STDOUT_DEBUG=1
   amp "hello"
   # Check logs for system prompt content
   # Does it mention todo tools?
   ```

**Results to Document:**
- Does setting disable todo tools? (YES/NO)
- Does system prompt still mention todos when disabled? (YES/NO)
- Confirm if setting is hardcoded to true

---

### Test 4: `amp.tools.inactivityTimeout`

**Status:** UNVERIFIED  
**Hypothesis:** Controls timeout for inactive tools

**Test Setup:**
```json
{
  "amp.tools.inactivityTimeout": 10
}
```

**Test Steps:**

1. **Baseline** (default 300 seconds):
   ```bash
   amp "run a command that sleeps: sleep 20"
   ```
   Expected: Command runs for 20 seconds, completes normally

2. **Test** (timeout set to 10 seconds):
   ```bash
   amp "run a command that sleeps: sleep 20"
   ```
   Expected if works: Command times out after 10 seconds
   Expected if broken: Command still runs full 20 seconds

3. **Alternative test with `amp.tools.stopTimeout`**:
   Test if this is actually the same setting under different name:
   ```json
   {
     "amp.tools.stopTimeout": 10
   }
   ```
   Repeat test. Does behavior match?

**Results to Document:**
- Does `inactivityTimeout` cause early timeout? (YES/NO)
- Does `stopTimeout` cause early timeout? (YES/NO)
- Are they the same setting? (YES/NO)

---

### Test 5: `amp.debug.httpLogging`

**Status:** UNVERIFIED  
**Hypothesis:** Enables HTTP request/response logging

**Test Setup:**
```json
{
  "amp.debug.httpLogging": true
}
```

**Test Steps:**

1. **Baseline** (setting=false or not set):
   ```bash
   export AMP_LOG_LEVEL=debug
   export AMP_LOG_FILE=~/.amp/test.log
   amp "hello"
   
   # Check logs for HTTP details
   grep -i "http\|request\|response" ~/.amp/test.log
   ```
   Document level of HTTP logging

2. **Test** (setting=true):
   ```bash
   # Clean log file
   rm ~/.amp/test.log
   
   export AMP_LOG_LEVEL=debug
   export AMP_LOG_FILE=~/.amp/test.log
   amp "hello"
   
   # Check logs for HTTP details
   grep -i "http\|request\|response" ~/.amp/test.log
   ```
   Expected if works: Much more detailed HTTP logging (headers, bodies, etc.)

**Results to Document:**
- Does setting increase HTTP logging detail? (YES/NO)
- What additional info is logged when enabled?
- Are request bodies/headers visible? (YES/NO)

---

### Test 6: Environment Variable `AMP_PWD`

**Status:** UNVERIFIED  
**Hypothesis:** Overrides working directory

**Test Setup:**
```bash
export AMP_PWD=/tmp
```

**Test Steps:**

1. **Baseline** (no env var):
   ```bash
   cd /home/user
   amp "what is the current directory?"
   ```
   Expected: Agent reports `/home/user` (or uses relative paths from there)

2. **Test** (AMP_PWD set):
   ```bash
   cd /home/user
   export AMP_PWD=/tmp
   amp "what is the current directory?"
   ```
   Expected if works: Agent reports `/tmp`
   Expected if broken: Agent still reports `/home/user`

**Results to Document:**
- Does AMP_PWD override working directory? (YES/NO)
- Does it affect file operations? (YES/NO)

---

## Reporting Results

### Success Template

```markdown
## Test Results: amp.setting.name

**Tester:** Your Name  
**Date:** 2025-01-21  
**AMP Version:** 0.0.1761153678-gfa55cf  
**Platform:** macOS / Linux / Windows

### Result: ✅ VERIFIED / ❌ NOT WORKING / ⚠️ PARTIAL

**Baseline Behavior:**
[Describe what happened without the setting]

**Test Behavior:**
[Describe what happened with non-default setting]

**Log Evidence:**
```
[Paste relevant log excerpts showing setting being read]
```

**Conclusion:**
Setting works as documented / Setting does not work / Setting partially works

**Additional Notes:**
[Any edge cases, warnings, or unexpected behavior]
```

### Where to Report

1. **Update verification status document** - Edit `settings-verification-status.md`
2. **File GitHub issue** - Report to Amp CLI team if setting doesn't work
3. **Share with community** - Post in Amp community channels

---

## Automated Testing Script

Future improvement: Create automated test script:

```bash
#!/bin/bash
# test-settings.sh

# Test amp.notifications.enabled
test_notifications() {
  echo '{"amp.notifications.enabled": false}' > ~/.amp/settings.json
  # ... test logic
}

# Test amp.tools.disable
test_tools_disable() {
  echo '{"amp.tools.disable": ["Bash"]}' > ~/.amp/settings.json
  # ... test logic
}

# Run all tests
test_notifications
test_tools_disable
# ... more tests

# Restore backup
cp ~/.amp/settings.json.backup ~/.amp/settings.json
```

---

## Safety Notes

1. **Always backup** settings before testing
2. **Test in safe directory** - Don't test in production code
3. **Monitor tool execution** - Some tests involve running commands
4. **Resource limits** - Timeout tests may consume resources
5. **Restore settings** - Return to known-good config after testing

---

## Contributing

Help improve the documentation by testing unverified settings!

1. Follow testing methodology above
2. Document results clearly
3. Submit findings via GitHub issue or PR
4. Include platform/version info

**Current priority tests:**
- `amp.todos.enabled` (high priority - may be broken)
- `amp.notifications.enabled` (medium priority)
- `amp.tools.disable` (medium priority)
- `amp.tools.inactivityTimeout` (low priority)

---

## Version History

### v0.0.1761153678-gfa55cf (2025-01-21)
- Initial testing guide
- 6 test procedures documented
- Template for reporting results

---

**Need Help?**
- Consult [Settings Verification Status](./settings-verification-status.md) for current status
- Check [settings.md](./settings.md) for setting descriptions
- Enable debug logging for troubleshooting
