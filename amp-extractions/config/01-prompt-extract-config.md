# AMP CLI Configuration Extraction Request

I need you to extract and document all API endpoints and configuration settings from the AMP CLI minified JavaScript file located at `node_modules/@sourcegraph/amp/dist/main.js`.

You should always use the "newest" or "latest" installation of the amp cli from the `npm-packages/` directory. For example, `npm-packages/0.0.1761153678-gfa55cf/node_modules/@sourcegraph/amp/dist/main.js` if `0.0.1761153678-gfa55cf` is the newest version of the amp cli in `npm-packages/`

This extraction creates comprehensive reference documentation for:
1. All HTTP endpoints the CLI uses (for proxy configuration)
2. All configuration settings, both documented and undocumented (for advanced configuration)

---

## Required Outputs

Create or update the following files in `amp-extractions/config/`:

### 1. API Endpoints Reference
**File:** `/home/code/projects/ben-vargas/ai-amp-cli/main/amp-extractions/config/endpoints.md`

A complete reference of ALL HTTP endpoints used by the AMP CLI, organized by category.

**Required Sections:**

#### Model Inference Endpoints
Document all model provider endpoints that the CLI routes through `amp.url`:

- **Anthropic** (`/api/provider/anthropic`)
  - List all Claude models
  - SDK initialization code
  - Provider setting (`amp.anthropic.provider` for vertex routing)
  
- **OpenAI** (`/api/provider/openai/v1`)
  - List all GPT models
  - Note: Oracle uses Responses API, not Chat Completions
  
- **Google Vertex AI** (`/api/provider/google`)
  - List all Gemini models
  
- **xAI** (`/api/provider/xai/v1`)
  - List all Grok models
  
- **Cerebras** (`/api/provider/cerebras`)
  - Qwen models
  
- **Groq** (`/api/provider/groq`)
  - List models
  
- **Supernova** (`/api/provider/supernova/v1`)
  - List models
  
- **OpenRouter** (Direct connection - NOT proxied)
  - `https://openrouter.ai/api/v1`
  - Note which models use this (Kimi K2, Sonoma Sky)
  - Requirement: Must set `amp.openrouter.apiKey` or `OPENROUTER_API_KEY`

For each provider, include:
- Endpoint path
- Model list with descriptions
- SDK used (with code example)
- Source line number from main.js
- Any special configuration notes

#### User & Thread Management Endpoints
Document all endpoints for user data and thread operations:

- `GET /api/user` - User information
- `GET /api/threads?createdByUserID={id}` - List threads
- `GET /api/threads/{id}` - Get specific thread
- `POST /api/internal/uploadThread` - Upload/sync thread data

Include source line numbers and purpose for each.

#### GitHub Integration Endpoints
Document Librarian agent's GitHub access:

- `GET /api/internal/github-auth-status` - Check GitHub authentication
- `ALL /api/internal/github-proxy/{path}` - Proxy GitHub REST API calls
  - Include examples of proxied paths (search/commits, repos/*/contents, etc.)

#### Internal Service Endpoints
Document endpoints for account features:

- `GET /api/internal/getUserFreeTierStatus` - Free tier quota
- `POST /api/internal/updateUserTrainingMode` - Training data preference
- `POST /api/internal/recordAdEvent` - Ad tracking

Note: These may be inferred from method names if exact paths aren't in bundle.

#### Real-Time Connection Endpoints
Document SSE connections:

- `SSE /api/connect` - Amp Connect for web interface sync
  - List event types: user-message, approve-tool, reject-tool, disconnect

#### Telemetry & Metrics Endpoints
Document any telemetry endpoints found:

- Search for `/api/otel/v1/metrics` or similar
- Note if NOT found in bundle

#### Summary Tables
Provide tables summarizing:
- Inference endpoints (proxied through amp.url)
- Non-proxied endpoints (OpenRouter)
- Internal API endpoints with methods and purposes

#### Request Headers
Document required headers for:
- Proxied requests (Authorization, amp-application, amp-message-id, etc.)
- OpenRouter requests

#### Proxy Configuration Requirements
Provide clear guidance for intermediary proxy setup:
- What must be proxied to amp.url
- What must pass through directly (OpenRouter)
- Required headers to forward

---

### 2. Settings Reference
**File:** `/home/code/projects/ben-vargas/ai-amp-cli/main/amp-extractions/config/settings.md`

A complete reference of ALL Amp CLI configuration settings, split into documented and undocumented sections.

**Required Sections:**

#### Documented Settings
Extract from official manual at https://ampcode.com/manual#configuration

Organize by category:
- **General**: Core settings like `amp.permissions`, `amp.anthropic.thinking.enabled`
- **Git Integration**: `amp.git.commit.*` settings
- **Tools & Execution**: `amp.todos.enabled`, `amp.tools.disable`, `amp.tools.stopTimeout`
- **Terminal**: `amp.terminal.commands.nodeSpawn.loadProfile`
- **MCP Integration**: `amp.mcpServers`
- **Editor Only**: Settings for VS Code integration
- **CLI Only**: Settings for command-line interface

For each setting include:
- Full key name (e.g., `amp.git.commit.coauthor.enabled`)
- Type (boolean, string, number, array, object)
- Default value
- Description (from manual)
- Source reference (Manual URL)
- Examples where applicable

#### Undocumented Settings
Extract from code (settings registry, typically around main.js:5653)

Organize by category:
- **Core**: `amp.url` and other fundamental settings
- **Anthropic**: Provider, temperature, interleaved thinking
- **Model Configuration**: Oracle model, agent modes, visible modes
- **Experimental Agent Modes**: Complete list with descriptions
  - Production modes: smart, fast, free
  - Experimental modes: geppetto, pinocchio, claudius, opus4.1, bolt, hydrogen, kashmir, gossip, sonomoon, gronk-fast
  - Variants: :main, :search, :main+search
- **Terminal & Input**: Submit on enter, detached logs
- **Tab Completion**: All tab.* settings (mark as Editor-only)
- **Tools & Execution**: Inactivity timeout
- **Experimental Features**: JavaScript execution, native secrets, review tool
- **Guidance Files**: Spec file reading
- **Hooks System**: Custom hook support
- **OpenRouter Integration**: API key
- **JetBrains Integration**: Skip install flag
- **Deprecated Settings**: Legacy settings with migration notes

For each setting include:
- Full key name
- Type
- Default value (from code)
- Visibility flag (Hidden, Experimental, Deprecated)
- Description
- Source line number from main.js
- Migration notes if deprecated

#### Environment Variables
Document both documented and undocumented environment variables:

**Documented** (from CLI help output, main.js:5680):
- `AMP_API_KEY`
- `AMP_URL`
- `AMP_LOG_LEVEL`
- `AMP_LOG_FILE`
- `AMP_SETTINGS_FILE`
- Standard Node.js: `HTTP_PROXY`, `HTTPS_PROXY`, `NODE_EXTRA_CA_CERTS`

**Undocumented** (from code analysis):
- `AMP_CLI_STDOUT_DEBUG` - Debug logging
- `AMP_DEBUG` - Error detail
- `AMP_SKIP_UPDATE_CHECK` - Disable updates
- `AMP_HOME` - Install root
- `AMP_VERSION` - Pin version
- `AMP_TEST_UPDATE_STATUS` - Testing
- `OPENROUTER_API_KEY` - OpenRouter auth
- `AMP_TOOLBOX` - JetBrains integration
- `TOOLBOX_ACTION` - JetBrains protocol

For each, include type, default, description, and source line.

#### Configuration Examples
Provide practical examples:
- Minimal configuration
- Advanced configuration (multiple settings)
- OpenRouter model usage
- Native secrets storage
- Permission configuration
- MCP server setup

#### Notes on Settings
Document:
- Visibility flags (Documented, Hidden, Experimental, Deprecated)
- Setting scopes (CLI Only, Editor Only, Both)
- Migration warnings (deprecated → new)

#### Related Resources
Link to:
- Official Amp Manual
- Configuration guide
- Agent modes documentation
- Model endpoints reference (endpoints.md)

---

## Extraction Guidelines

### Use Oracle for Verification
1. **Always use the Oracle tool** to verify extracted information
2. Oracle should:
   - Scan the main.js bundle for endpoint URLs
   - Locate settings registry (typically CQ6 around line 5653)
   - Cross-reference with official documentation
   - Identify source line numbers for all findings
   - Validate defaults and types

### Accuracy Requirements
1. **Preserve exact values** - Don't approximate defaults or types
2. **Include source line numbers** - All findings must reference main.js line numbers
3. **Verify against manual** - Cross-check documented settings with https://ampcode.com/manual#configuration
4. **Complete extraction** - Don't omit settings or endpoints
5. **Track changes** - Note any discrepancies between code and documentation

### Formatting Standards
1. **Use consistent Markdown** - Headings, tables, code blocks
2. **Code examples** - Include SDK initialization code for providers
3. **Tables for summaries** - Endpoint tables, settings summaries
4. **Clear categorization** - Group related items
5. **Cross-references** - Link between related sections

---

## Process

### Step 1: Locate Source File
```bash
glob "**/node_modules/@sourcegraph/amp/dist/main.js"
```

### Step 2: Extract Endpoints (Use Oracle)
**Oracle Task**: "Find all HTTP endpoint URLs used by the AMP CLI"

Oracle should search for:
- `new URL("/api/provider/`, `baseURL` configurations
- Provider SDK initialization (Anthropic, OpenAI, Google, xAI, Cerebras, Groq, Supernova)
- OpenRouter direct connections (`openrouter.ai`)
- Internal API calls (`/api/user`, `/api/threads`, `/api/internal/*`)
- GitHub proxy (`/api/internal/github-proxy/`, `/api/internal/github-auth-status`)
- SSE connections (`/api/connect`)
- Telemetry endpoints (`/api/otel/`)
- API client methods (`uploadThread`, `getUserFreeTierStatus`, `updateUserTrainingMode`, `recordAdEvent`)

For each endpoint found:
- Record exact path
- Note source line number
- Identify HTTP method (GET, POST, SSE, etc.)
- Determine purpose
- Check if proxied through amp.url or direct connection

### Step 3: Extract Settings (Use Oracle)
**Oracle Task**: "Find all configuration settings in the CLI settings registry and env variables"

Oracle should search for:
- Settings registry object (typically `CQ6` around line 5653)
- Settings with `visible: false` or `hidden` flags
- Default values in registry
- Environment variable references (`process.env.AMP_*`, `process.env.OPENROUTER_API_KEY`)
- Experimental flags (`amp.experimental.*`)
- Internal settings (`amp.internal.*`)
- Deprecated settings with migration code

For each setting found:
- Record full key path
- Extract type and default value
- Note visibility flag
- Identify source line number
- Find description or usage context

### Step 4: Cross-Reference with Manual
**Oracle Task**: "Read https://ampcode.com/manual#configuration and list all documented settings"

Compare documented settings with code findings:
- Mark settings as "Documented" if in manual
- Mark as "Undocumented" if only in code
- Note any discrepancies in defaults
- Identify deprecated settings

### Step 5: Organize and Format
Create two markdown files:

**endpoints.md**:
- Header with version info
- Model inference endpoints (7 providers + OpenRouter)
- User & thread endpoints
- GitHub endpoints
- Internal service endpoints
- Real-time endpoints
- Telemetry endpoints
- Summary tables
- Request headers
- Proxy configuration guide

**settings.md**:
- Table of contents
- Documented settings (organized by category)
- Undocumented settings (organized by category)
- Environment variables (documented + undocumented)
- Configuration examples
- Notes on settings
- Related resources

### Step 6: Add Metadata
Both files should include:
- Generation source: CLI version
- Verification: Oracle analysis reference
- Last updated: Date
- Source file: main.js with line references

---

## Success Criteria

### For endpoints.md
- ✅ All 7+ model provider endpoints documented with source lines
- ✅ OpenRouter direct connection clearly marked as NOT proxied
- ✅ All internal API endpoints identified (user, threads, GitHub, services)
- ✅ SSE connection documented with event types
- ✅ Request headers documented for proxied and direct requests
- ✅ Summary tables provided
- ✅ Proxy configuration requirements clearly stated
- ✅ All findings verified by Oracle with line numbers

### For settings.md
- ✅ All documented settings listed with manual references
- ✅ All undocumented settings extracted from code with line numbers
- ✅ Settings properly categorized (documented vs undocumented)
- ✅ Correct types and defaults for all settings
- ✅ Environment variables comprehensively documented
- ✅ Configuration examples provided
- ✅ Migration warnings for deprecated settings
- ✅ Cross-references to related documentation
- ✅ All findings verified by Oracle

### General Quality
- ✅ Markdown formatting is clean and consistent
- ✅ Tables are properly formatted
- ✅ Code blocks have language tags
- ✅ All source line numbers provided
- ✅ No approximations or guesses - all data verified
- ✅ Clear categorization and organization
- ✅ Comprehensive enough for production use

---

## Verification Checklist

Before completing the extraction, verify:

1. **Endpoints completeness**:
   - [ ] All 7 main providers documented
   - [ ] OpenRouter special case noted
   - [ ] All /api/user, /api/threads endpoints found
   - [ ] GitHub proxy endpoints documented
   - [ ] Internal service endpoints identified
   - [ ] SSE connection documented
   - [ ] Source lines for all endpoints

2. **Settings completeness**:
   - [ ] Manual settings cross-referenced
   - [ ] Hidden settings from registry extracted
   - [ ] Experimental settings identified
   - [ ] Deprecated settings noted with migrations
   - [ ] Environment variables documented
   - [ ] Source lines for all settings
   - [ ] Defaults verified against code

3. **Oracle verification**:
   - [ ] Oracle consulted for endpoint discovery
   - [ ] Oracle consulted for settings registry
   - [ ] Oracle verified against manual
   - [ ] Oracle findings documented with line numbers
   - [ ] Discrepancies resolved and noted

4. **Documentation quality**:
   - [ ] Markdown properly formatted
   - [ ] Tables render correctly
   - [ ] Code blocks have language tags
   - [ ] Cross-references work
   - [ ] Examples are practical
   - [ ] Metadata included (version, date, source)

---

## Additional Context

### Purpose
These configuration documents serve multiple purposes:
1. **Proxy Configuration**: Administrators can configure intermediary proxies to route Amp CLI traffic correctly
2. **Advanced Configuration**: Users can leverage undocumented settings for specific use cases
3. **Debugging**: Developers can understand the full configuration surface area
4. **Migration**: Users can identify deprecated settings and migrate to new ones

### Scope
- **endpoints.md**: Focus on HTTP communication - what the CLI calls and where
- **settings.md**: Focus on configuration - what the CLI reads from config files and environment

### Maintenance
When running this extraction on a new version of the CLI:
1. Update the version number in metadata
2. Check for new providers or endpoints
3. Look for new settings or deprecated settings
4. Verify defaults haven't changed
5. Update source line numbers
6. Re-run Oracle verification
7. Update "Last Updated" date

### Known Edge Cases
- Some endpoint paths may be inferred from method names (uploadThread, recordAdEvent) if not in bundle as string literals
- OpenTelemetry endpoint may not be found if not explicitly called
- Some settings may have different defaults in code vs documentation (note discrepancies)
- Tab completion settings are editor-specific, mark as such
- Environment variables may be referenced without explicit default values

---

## Example Oracle Prompts

### For Endpoint Discovery
```
Review the minified main.js file and identify ALL HTTP endpoints used by the AMP CLI.

Specifically look for:
1. Model provider baseURL configurations (Anthropic, OpenAI, Google, xAI, Cerebras, Groq, Supernova)
2. OpenRouter direct connections
3. Internal API endpoints (/api/user, /api/threads, /api/internal/*)
4. GitHub integration (/api/internal/github-proxy/*, /api/internal/github-auth-status)
5. SSE connections
6. Telemetry endpoints

For each endpoint found, provide:
- Exact path or pattern
- Source line number
- HTTP method
- Purpose
- Whether proxied through amp.url or direct connection

Be exhaustive - we need this for proxy configuration.
```

### For Settings Discovery
```
Review the minified main.js file and extract ALL configuration settings.

Specifically look for:
1. Settings registry (typically around line 5653, variable CQ6)
2. Settings with visible:false or hidden flags
3. Environment variable references (process.env.AMP_*, etc.)
4. Experimental settings (amp.experimental.*)
5. Internal settings (amp.internal.*)
6. Deprecated settings

For each setting found, provide:
- Full key path
- Type (boolean, string, number, array, object)
- Default value from code
- Visibility flag
- Source line number
- Any description or usage context

Also cross-reference with https://ampcode.com/manual#configuration to identify which settings are documented vs undocumented.
```

### For Verification
```
Verify the accuracy of the extracted configuration data:

1. Check all endpoint paths are correct
2. Verify all setting defaults match the code
3. Cross-reference documented settings with manual
4. Identify any missing endpoints or settings
5. Validate source line numbers
6. Note any discrepancies

Provide a verification report with:
- Confirmed correct items
- Items needing correction
- Missing items
- Discrepancies between code and documentation
```

---

**Template Version**: 1.0  
**Compatible with**: Amp CLI v0.0.1760000000+ (minified dist/main.js)  
**Oracle-Assisted**: Yes (required for verification)  
**Output Files**: `endpoints.md`, `settings.md`
