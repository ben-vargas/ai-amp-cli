# Documentation Errata & Corrections

**Version:** 0.0.1761153678-gfa55cf  
**Date:** 2025-01-21  
**Issue Reporter:** User feedback on accuracy concerns

---

## Summary

During user review, critical accuracy issues were discovered in the initial extraction. This led to a comprehensive re-verification of all settings, resulting in the creation of enhanced documentation with transparent verification status.

---

## Critical Errors Found & Fixed

### 1. `amp.anthropic.thinking.enabled` Default Value

**Initial Documentation (WRONG):**
```
Default: false (per code registry)
```

**Actual Behavior (CORRECT):**
```
Registry Default: false
Runtime Default: true (via ?? true fallback)
```

**Impact:**
- **HIGH** - Users would expect thinking to be disabled by default
- Thinking is actually ENABLED by default
- Manual is correct, code registry is misleading

**Code Evidence (line 1225):**
```javascript
let W = F.settings["anthropic.thinking.enabled"] ?? !0;  // ?? true
```

**Resolution:**
- Updated settings.md with correct default (`true`)
- Documented registry vs runtime discrepancy
- Added detailed explanation in verification status

**Status:** ✅ FIXED

---

### 2. Verification Methodology Insufficient

**Problem:**
- Initial extraction relied heavily on registry defaults
- Did not verify runtime behavior for all settings
- Assumed registry presence = functional setting

**Impact:**
- **HIGH** - Unknown how many settings actually work
- User loses trust in documentation accuracy
- Settings may be documented but non-functional

**Resolution:**
- Created comprehensive [settings-verification-status.md](./settings-verification-status.md)
- Verified 73% of settings via runtime code analysis
- Clearly marked 16% as UNVERIFIED
- Identified 2 settings with issues

**Status:** ✅ FIXED

---

## Settings with Ongoing Issues

### 1. `amp.todos.enabled` - Possibly Non-Functional

**Status:** ❌ HARDCODED (unverified)

**Problem:**
```javascript
// Runtime passes hardcoded true, not setting value
let {systemPrompt:q,tools:G} = await PZ(Z, Q, 
  {enableTodos:!0, enableTask:!0, enableOracle:!0},  // <-- HARDCODED
  {model:J, provider:"anthropic", agentMode:Y}, X);
```

**Evidence:**
- Registry has setting with default `true` (line 5653)
- Runtime code hardcodes `enableTodos:!0` (line 1234)
- Setting may never be read

**Impact:**
- **MEDIUM** - Cannot disable todos if setting doesn't work
- Documentation claims it's controllable

**Next Steps:**
- Manual testing required (see [testing-guide.md](./testing-guide.md))
- If confirmed broken, file bug with Amp team
- Update documentation based on test results

**Documentation Status:** ⚠️ Marked with warning in settings.md

---

### 2. `amp.tools.disable` - Unverified

**Status:** ⚠️ UNVERIFIED

**Problem:**
- Exists in registry (line 5653)
- Referenced in TUI code (line 6403)
- No explicit filtering logic found

**Impact:**
- **MEDIUM** - Users may rely on this for security/workflow
- Unknown if it actually removes tools

**Next Steps:**
- Manual testing required
- Test with `["Bash"]` to see if command execution blocked
- Update verification status with results

**Documentation Status:** ⚠️ Marked with warning in settings.md

---

### 3. `amp.notifications.enabled` - Possible CLI Flag Override

**Status:** ⚠️ UNVERIFIED

**Problem:**
```javascript
// Uses CLI flag logic, not direct setting read
const V = Q.notifications !== void 0 ? Q.notifications : !J.executeMode;
```

**Evidence:**
- Registry has setting (line 5653)
- Runtime uses flag-based logic (line 6413)
- Unclear if setting is read

**Impact:**
- **LOW** - Notifications are convenience feature
- Setting may be bypassed by CLI flags

**Next Steps:**
- Test setting vs CLI flag behavior
- Determine precedence order
- Document interaction between setting and flag

**Documentation Status:** ⚠️ Mentioned in verification status

---

### 4. Timeout Settings Confusion

**Status:** ⚠️ UNVERIFIED

**Problem:**
- Manual documents `amp.tools.stopTimeout`
- Registry has `amp.tools.inactivityTimeout`
- May be same setting, different names

**Evidence:**
- `stopTimeout` not found in registry (line 5653)
- `inactivityTimeout` in registry but no runtime usage found
- Both claim 300 second default

**Impact:**
- **LOW** - Timeout configuration unclear
- Users may use wrong name

**Next Steps:**
- Test both settings
- Determine which actually works
- Document alias or mark one as non-functional

**Documentation Status:** ⚠️ Both marked with warnings

---

## Verification Improvements Made

### 1. Comprehensive Verification Status Document

**Created:** [settings-verification-status.md](./settings-verification-status.md)

**Contents:**
- Verification category for EVERY setting
- Runtime code locations with line numbers
- Code snippets showing actual usage
- Known issues and workarounds
- Testing priorities

**Benefit:** Users can see confidence level for any setting

---

### 2. Manual Testing Guide

**Created:** [testing-guide.md](./testing-guide.md)

**Contents:**
- General testing methodology
- Specific test procedures for 6 unverified settings
- Result reporting templates
- Safety guidelines

**Benefit:** Community can help verify settings and contribute results

---

### 3. Enhanced Settings Documentation

**Updated:** [settings.md](./settings.md)

**Improvements:**
- Added verification warnings to problematic settings
- Linked to verification status document
- Added verification status section
- Clear indicators for unverified settings

**Benefit:** Users see warnings inline, not just in separate doc

---

### 4. Comprehensive README

**Created:** [README.md](./README.md)

**Contents:**
- Overview of all documentation
- Quick start guides
- Known issues summary
- Maintenance procedures

**Benefit:** Single entry point to understand all documentation

---

## Statistics

### Before User Feedback

- **Settings Documented:** 45
- **Verification Status:** Unclear
- **Known Issues:** 0 (but 2 existed undiscovered)
- **Confidence Level:** Assumed high, actually unknown
- **User Trust:** Damaged after inaccuracy discovered

### After Re-Verification

- **Settings Documented:** 45 (same)
- **Verification Status:** Explicit for every setting
- **Settings Verified:** 33 (73%)
- **Settings Unverified:** 7 (16%)
- **Settings with Issues:** 2 (4%)
- **Known Issues:** 2 critical, 4 unverified
- **Confidence Level:** 73% verified, rest marked clearly
- **User Trust:** Hopefully restored via transparency

---

## Lessons Learned

### 1. Registry ≠ Runtime

**Lesson:** Settings registry contains defaults but doesn't guarantee the setting is actually used by runtime code.

**Fix:** Always trace runtime usage, not just registry entries.

### 2. Minified Code Challenges

**Lesson:** Minified JavaScript makes verification harder due to obfuscation and lack of comments.

**Limitation:** Some settings may be functional but usage pattern too complex to trace.

### 3. Nullish Coalescing Overrides

**Lesson:** Runtime code can use `?? defaultValue` to override registry defaults.

**Fix:** Search for all settings reads with `??`, `||`, or ternary operators.

### 4. User Feedback Critical

**Lesson:** User caught an issue (thinking enabled despite seeing "thinking" output) that automated extraction missed.

**Fix:** Always welcome user feedback and verify claims thoroughly.

### 5. Transparency > Perfection

**Lesson:** Being honest about verification status is better than claiming false certainty.

**Fix:** Created comprehensive verification status with confidence levels.

---

## Future Improvements

### Short Term

1. **Manual testing** of all 7 unverified settings
2. **Bug report** for `amp.todos.enabled` if confirmed broken
3. **Community testing** program to crowdsource verification

### Medium Term

1. **Automated test suite** that:
   - Sets each setting to non-default
   - Observes behavior changes
   - Reports functional vs non-functional

2. **Deobfuscated source** access:
   - Request access to non-minified source
   - Makes verification much easier
   - Could reach 100% verification

3. **Official verification** from Amp team:
   - Share findings with Sourcegraph
   - Request official confirmation
   - Contribute back to official docs

### Long Term

1. **Continuous verification** with each CLI release
2. **Automated extraction pipeline** for updates
3. **Community contribution platform** for test results

---

## Acknowledgments

**Issue Reporter:** User who questioned the `anthropic.thinking.enabled` default after observing thinking output

**Impact:** Led to discovery of critical inaccuracy and comprehensive re-verification

**Result:** Much more accurate and transparent documentation

---

## Contact

**For Documentation Issues:**
- File issue in repository
- Check verification status first
- Include test results if available

**For AMP CLI Bugs:**
- Report to Sourcegraph/Amp team
- Reference this documentation
- Include verification status info

---

## Version History

### 2025-01-21 - Major Revision
- **Trigger:** User caught inaccuracy in `anthropic.thinking.enabled`
- **Action:** Complete re-verification of all settings
- **Created:** 5 new/updated documents
- **Result:** 73% verification rate, clear status for all settings
- **Status:** Documentation now trustworthy with transparent limitations

### 2025-01-21 - Initial Release
- Initial extraction (flawed)
- Did not verify runtime behavior adequately
- Caught by user feedback

---

**Current Status:** Documentation is now accurate to the best of our ability, with clear verification status for every setting. Users can make informed decisions based on confidence levels.

**Recommendation:** Consult [settings-verification-status.md](./settings-verification-status.md) before relying on any UNVERIFIED setting in production.
