# Amp CLI API Endpoints Reference

Complete reference of all HTTP endpoints used by the Amp CLI for model inference, user management, telemetry, and internal operations.

**Base URL**: All proxied endpoints use `amp.url` setting (default: `https://ampcode.com`)

---

## Model Inference Endpoints

These endpoints handle AI model inference requests, routed through the Amp proxy server.

### Anthropic (Claude Models)

**Endpoint**: `{amp.url}/api/provider/anthropic`

**Models**:
- `claude-sonnet-4-20250514` (Claude Sonnet 4)
- `claude-sonnet-4-5-20250929` (Claude Sonnet 4.5) - **Default for Smart mode**
- `claude-opus-4-20250514` (Claude Opus 4)
- `claude-opus-4-1-20250805` (Claude Opus 4.1)
- `claude-3-5-haiku-20241022` (Claude 3.5 Haiku)
- `claude-3-5-sonnet-20241022` (Claude 3.5 Sonnet)

**SDK**: Anthropic SDK
**Source**: `node_modules/@sourcegraph/amp/dist/main.js:1210`

```javascript
new Anthropic({
  apiKey: apiKey,
  baseURL: new URL("/api/provider/anthropic", ampURL).toString()
})
```

**Provider Setting**: `amp.anthropic.provider` controls whether to use `"anthropic"` (default) or `"vertex"` for Claude models.

---

### OpenAI (GPT Models)

**Endpoint**: `{amp.url}/api/provider/openai/v1`

**Models**:
- `gpt-5` (GPT-5) - **Default Oracle model**
- `gpt-5-codex` (GPT-5 Codex)
- `gpt-5-mini` (GPT-5 Mini)
- `gpt-5-nano` (GPT-5 Nano)
- `gpt-4.1` (GPT-4.1)
- `gpt-4.1-mini` (GPT-4.1 Mini)
- `gpt-4.1-nano` (GPT-4.1 Nano)
- `gpt-4o` (GPT-4o)
- `gpt-4o-mini` (GPT-4o Mini)
- `o3` (o3)
- `o3-mini` (o3-mini)
- `o4-mini-deep-research` (o4 Mini Deep Research)
- `o3-deep-research` (o3 Deep Research)

**SDK**: OpenAI SDK (`openai` npm package)
**Source**: `node_modules/@sourcegraph/amp/dist/main.js:1930`

```javascript
new OpenAI({
  apiKey: apiKey,
  baseURL: new URL("/api/provider/openai/v1", ampURL).toString()
})
```

**Special Note**: Oracle tool uses OpenAI **Responses API** endpoint (`/v1/responses`), not Chat Completions.

---

### Google Vertex AI (Gemini Models)

**Endpoint**: `{amp.url}/api/provider/google`

**Models**:
- `gemini-2.5-pro` (Gemini 2.5 Pro)
- `gemini-2.5-flash` (Gemini 2.5 Flash)
- `gemini-2.5-flash-preview-09-2025` (Gemini 2.5 Flash Preview)
- `gemini-2.5-flash-lite` (Gemini 2.5 Flash Lite)
- `gemini-2.5-flash-lite-preview-09-2025` (Gemini 2.5 Flash Lite Preview) - **Default for Free mode**

**SDK**: Google Generative AI SDK
**Source**: `node_modules/@sourcegraph/amp/dist/main.js:1295`

```javascript
new GoogleGenerativeAI({
  apiKey: "placeholder",
  vertexai: true,
  googleAuthOptions: {},
  httpOptions: {
    baseUrl: new URL("/api/provider/google", ampURL).toString(),
    headers: { Authorization: "Bearer " + apiKey }
  }
})
```

---

### xAI (Grok Models)

**Endpoint**: `{amp.url}/api/provider/xai/v1`

**Models**:
- `grok-code-fast-1` (Grok Code Fast 1) - **Default for Fast mode**
- `grok-code` (Grok Code) - **Used in Free mode**

**SDK**: OpenAI SDK (xAI API is OpenAI-compatible)
**Source**: `node_modules/@sourcegraph/amp/dist/main.js:1939`

```javascript
new OpenAI({
  apiKey: apiKey,
  baseURL: new URL("/api/provider/xai/v1", ampURL).toString()
})
```

---

### Cerebras (Qwen Models)

**Endpoint**: `{amp.url}/api/provider/cerebras`

**Models**:
- `qwen-3-coder-480b` (Qwen 3 Coder 480B)

**SDK**: Custom Cerebras SDK
**Source**: `node_modules/@sourcegraph/amp/dist/main.js:1931`

```javascript
new CerebrasClient({
  apiKey: apiKey,
  baseURL: new URL("/api/provider/cerebras", ampURL).toString()
})
```

---

### Groq

**Endpoint**: `{amp.url}/api/provider/groq`

**Models**:
- `openai/gpt-oss-120b` (GPT OSS 120B)

**SDK**: OpenAI SDK (Groq API is OpenAI-compatible)
**Source**: `node_modules/@sourcegraph/amp/dist/main.js:1931`

```javascript
new OpenAI({
  apiKey: apiKey,
  baseURL: new URL("/api/provider/groq", ampURL).toString()
})
```

---

### Supernova

**Endpoint**: `{amp.url}/api/provider/supernova/v1`

**Models**:
- `code-supernova` (Code Supernova)

**SDK**: OpenAI SDK (Supernova API is OpenAI-compatible)
**Source**: `node_modules/@sourcegraph/amp/dist/main.js:1939`

```javascript
new OpenAI({
  apiKey: apiKey,
  baseURL: new URL("/api/provider/supernova/v1", ampURL).toString()
})
```

---

### OpenRouter (Direct Connection - NOT Proxied)

**Endpoint**: `https://openrouter.ai/api/v1` ⚠️ **Does NOT use amp.url**

**Models**:
- `kimi-k2-instruct-0905` (Kimi K2 Instruct) → sent as `"moonshotai/kimi-k2-instruct-0905"`
- `sonoma-sky-alpha` (Sonoma Sky Alpha) → sent as `"openrouter/sonoma-sky-alpha"`

**SDK**: OpenAI SDK
**Source**: `node_modules/@sourcegraph/amp/dist/main.js:1938`

```javascript
new OpenAI({
  apiKey: config.settings["openrouter.apiKey"] || process.env.OPENROUTER_API_KEY,
  baseURL: "https://openrouter.ai/api/v1"
})
```

**Requirements**: 
- Must set `amp.openrouter.apiKey` or `OPENROUTER_API_KEY` environment variable
- Bypasses Amp proxy entirely

---

## User & Thread Management Endpoints

These endpoints manage user authentication, thread storage, and synchronization.

### User Information

**GET** `{amp.url}/api/user`

Retrieves current user information including ID, email, and account status.

**Source**: `node_modules/@sourcegraph/amp/dist/main.js:6469`

**Response**: User object with ID, email, subscription details

---

### Thread Listing

**GET** `{amp.url}/api/threads?createdByUserID={userID}`

Lists all threads created by the authenticated user.

**Source**: `node_modules/@sourcegraph/amp/dist/main.js:6470`

**Query Parameters**:
- `createdByUserID`: User ID to filter threads

**Response**: Array of thread objects

---

### Thread Retrieval

**GET** `{amp.url}/api/threads/{threadID}`

Fetches a specific thread by ID.

**Source**: `node_modules/@sourcegraph/amp/dist/main.js:6466`

**Response**: Thread object with messages and metadata

---

### Thread Upload/Synchronization

**Endpoint**: `{amp.url}/api/internal/uploadThread` (inferred)

Uploads/synchronizes thread data to the Amp server for web interface access.

**Source**: `node_modules/@sourcegraph/amp/dist/main.js:6464`

**Method**: Internal API client method `uploadThread`

**Note**: Exact path not visible in bundle; method referenced in synchronization flow.

---

## GitHub Integration Endpoints

These endpoints support the Librarian agent's GitHub code exploration capabilities.

### GitHub Authentication Status

**GET** `{amp.url}/api/internal/github-auth-status`

Checks if the user has authenticated with GitHub for Librarian access.

**Source**: `node_modules/@sourcegraph/amp/dist/main.js:5162`

**Response**: 
```json
{
  "authenticated": true/false
}
```

---

### GitHub Proxy

**ALL** `{amp.url}/api/internal/github-proxy/{githubPath}`

Proxies arbitrary GitHub REST API calls with authentication.

**Source**: `node_modules/@sourcegraph/amp/dist/main.js:4859`

**Examples**:
- `/api/internal/github-proxy/search/commits?q=...`
- `/api/internal/github-proxy/repos/{owner}/{repo}/commits`
- `/api/internal/github-proxy/repos/{owner}/{repo}/contents/{path}`
- `/api/internal/github-proxy/search/code?q=...`

**Purpose**: Allows Librarian to access GitHub API on behalf of user without exposing tokens to client.

---

## Internal Service Endpoints

These endpoints handle user status, advertisements, and account features.

### Free Tier Status

**Endpoint**: `{amp.url}/api/internal/getUserFreeTierStatus` (inferred)

Retrieves user's free tier quota and usage information.

**Source**: `node_modules/@sourcegraph/amp/dist/main.js:6411`

**Method**: Internal API client method `getUserFreeTierStatus`

**Response**: Free tier status including remaining quota

---

### Training Mode Update

**Endpoint**: `{amp.url}/api/internal/updateUserTrainingMode` (inferred)

Updates user's training data sharing preference.

**Source**: `node_modules/@sourcegraph/amp/dist/main.js:6416`

**Method**: Internal API client method `updateUserTrainingMode`

**Purpose**: Toggles whether user data is used for model training (free tier requirement)

---

### Advertisement Events

**Endpoint**: `{amp.url}/api/internal/recordAdEvent` (inferred)

Records advertisement impressions and interactions for free tier users.

**Source**: `node_modules/@sourcegraph/amp/dist/main.js:6412`

**Method**: Internal API client method `recordAdEvent`

**Purpose**: Tracks ad engagement for free tier monetization

---

## Real-Time Connection Endpoints

### Amp Connect (SSE)

**Endpoint**: `{amp.url}/api/connect` (inferred)

Server-Sent Events (SSE) connection for real-time web interface communication.

**Source**: `node_modules/@sourcegraph/amp/dist/main.js:4634`

**Purpose**: 
- Receive tool approval/rejection from web interface
- Sync messages between CLI and web
- Handle disconnect events

**Events**:
- `user-message`: New message from web interface
- `approve-tool`: Tool use approved
- `reject-tool`: Tool use rejected
- `disconnect`: User disconnected via web

---

## Telemetry & Metrics Endpoints

### OpenTelemetry Metrics

**Status**: NOT FOUND in bundle

**Expected Endpoint**: `{amp.url}/api/otel/v1/metrics`

**Note**: While referenced in documentation, the Oracle did not find explicit calls to this endpoint in the CLI bundle. Metrics may be sent through a different mechanism or endpoint.

---

## Summary Tables

### Inference Endpoints (Proxied through amp.url)

| Provider | Endpoint Path | Models Count | Source Line |
|----------|---------------|--------------|-------------|
| Anthropic | `/api/provider/anthropic` | 6 | 1210 |
| OpenAI | `/api/provider/openai/v1` | 13 | 1930 |
| Google Vertex AI | `/api/provider/google` | 5 | 1295 |
| xAI | `/api/provider/xai/v1` | 2 | 1939 |
| Cerebras | `/api/provider/cerebras` | 1 | 1931 |
| Groq | `/api/provider/groq` | 1 | 1931 |
| Supernova | `/api/provider/supernova/v1` | 1 | 1939 |

### Non-Proxied Endpoints

| Service | Endpoint | Purpose |
|---------|----------|---------|
| OpenRouter | `https://openrouter.ai/api/v1` | Kimi K2, Sonoma Sky models (direct) |

### Internal API Endpoints (Proxied through amp.url)

| Category | Endpoint | Method | Purpose | Source |
|----------|----------|--------|---------|--------|
| User | `/api/user` | GET | Get user info | 6469 |
| Threads | `/api/threads` | GET | List threads | 6470 |
| Threads | `/api/threads/{id}` | GET | Get thread | 6466 |
| Threads | `/api/internal/uploadThread` | POST | Upload thread | 6464 |
| GitHub | `/api/internal/github-auth-status` | GET | Check GitHub auth | 5162 |
| GitHub | `/api/internal/github-proxy/*` | ALL | Proxy GitHub API | 4859 |
| Account | `/api/internal/getUserFreeTierStatus` | GET | Free tier status | 6411 |
| Account | `/api/internal/updateUserTrainingMode` | POST | Update training mode | 6416 |
| Ads | `/api/internal/recordAdEvent` | POST | Record ad event | 6412 |
| Real-time | `/api/connect` | SSE | Web interface sync | 4634 |

---

## Request Headers

### Proxied Requests (through amp.url)

All proxied model inference and internal API requests include:

```javascript
{
  "Authorization": "Bearer " + apiKey,
  "amp-application": "amp.chat",
  "amp-message-id": messageId,    // when available
  "amp-thread-id": threadId,      // when available
  // Plus workspace/tree metadata
}
```

### OpenRouter Requests

```javascript
{
  "amp-application": "amp.chat",
  "HTTP-Referer": threadID,
  "x-grok-conv-id": threadID      // for xAI models
}
```

---

## Authentication Flow

1. **Retrieve API Key**: `secrets.getToken("apiKey", amp.url)`
2. **Construct Client**: Provider-specific SDK with appropriate `baseURL`
3. **Make Request**: SDK handles HTTP request to Amp proxy
4. **Proxy Routes**: Amp server at `amp.url` forwards to actual provider API
5. **Response**: Streamed back through proxy to CLI

**Exception**: OpenRouter connections bypass Amp proxy and connect directly to `openrouter.ai`.

---

## Proxy Configuration Requirements

For an intermediary proxy to properly route Amp CLI traffic:

### Must Proxy to amp.url:
- All `/api/provider/*` paths (model inference)
- All `/api/user` and `/api/threads/*` paths
- All `/api/internal/*` paths
- SSE connection at `/api/connect`

### Must Pass Through Directly:
- `https://openrouter.ai/api/v1` (OpenRouter models)

### Required Headers to Forward:
- `Authorization`
- `amp-application`
- `amp-message-id`
- `amp-thread-id`
- All workspace/tree metadata headers

---

**Generated from**: Amp CLI v0.0.1761076893-ge5520f source code analysis  
**Verified by**: Oracle analysis of `node_modules/@sourcegraph/amp/dist/main.js`  
**Last Updated**: 2025-01-20
