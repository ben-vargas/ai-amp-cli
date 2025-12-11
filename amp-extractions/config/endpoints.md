# AMP CLI API Endpoints Reference

**Version:** 0.0.1761153678-gfa55cf  
**Source:** `node_modules/@sourcegraph/amp/dist/main.js`  
**Last Updated:** 2025-01-21  
**Verification:** Oracle-assisted analysis

---

## Table of Contents

1. [Overview](#overview)
2. [Model Inference Endpoints](#model-inference-endpoints)
3. [User & Thread Management Endpoints](#user--thread-management-endpoints)
4. [GitHub Integration Endpoints](#github-integration-endpoints)
5. [Real-Time Connection Endpoints](#real-time-connection-endpoints)
6. [Telemetry & Metrics Endpoints](#telemetry--metrics-endpoints)
7. [Summary Tables](#summary-tables)
8. [Request Headers](#request-headers)
9. [Proxy Configuration Requirements](#proxy-configuration-requirements)

---

## Overview

This document catalogs all HTTP endpoints used by the AMP CLI for external communication. These endpoints fall into two categories:

- **Proxied through `amp.url`**: Most endpoints route through the configured Amp server (default: `https://ampcode.com`)
- **Direct connections**: Only OpenRouter bypasses the Amp proxy and connects directly

Understanding these endpoints is critical for:
- Configuring intermediary proxies
- Network security policies
- Debugging connection issues
- Understanding data flow

---

## Model Inference Endpoints

All model inference endpoints (except OpenRouter) are proxied through the configured `amp.url` setting. Each provider has a dedicated endpoint path that routes model requests through the Amp infrastructure.

### Anthropic (Claude)

**Endpoint:** `{amp.url}/api/provider/anthropic`  
**Source:** main.js:1225  
**Method:** POST  
**Proxied:** Yes (through amp.url)

**Models:**
- Claude 3.5 Sonnet
- Claude 3.5 Haiku
- Claude 3 Opus
- Claude 3 Sonnet
- Claude 3 Haiku

**SDK Used:** Anthropic SDK (`@anthropic-ai/sdk`)

**Configuration:**
```typescript
const client = new Anthropic({
  baseURL: `${ampUrl}/api/provider/anthropic`,
  apiKey: ampApiKey,
  // Additional headers set by CLI
});
```

**Special Features:**
- Supports streaming responses (SSE)
- Supports thinking mode (configured via `amp.anthropic.thinking.enabled`)
- Supports interleaved thinking (via `amp.anthropic.interleavedThinking.enabled`)
- Can route through Google Vertex AI (via `amp.anthropic.provider` setting)

**Notes:**
- Uses both `/messages` (Chat Completions) and `/responses` (Responses API) depending on context
- Thinking mode enables extended reasoning capabilities
- Vertex routing allows using Claude through Google Cloud infrastructure

---

### OpenAI (GPT)

**Endpoint:** `{amp.url}/api/provider/openai` or `{amp.url}/api/provider/openai/v1`  
**Source:** main.js:1930  
**Method:** POST  
**Proxied:** Yes (through amp.url)

**Models:**
- GPT-4 Turbo
- GPT-4
- GPT-3.5 Turbo
- o1 (reasoning models)

**SDK Used:** OpenAI SDK (`openai`)

**Configuration:**
```typescript
const client = new OpenAI({
  baseURL: `${ampUrl}/api/provider/openai/v1`,
  apiKey: ampApiKey,
});
```

**Special Features:**
- Supports streaming responses (SSE)
- Oracle agent uses Responses API instead of Chat Completions
- Compatible with standard OpenAI API structure

**Notes:**
- The Oracle tool specifically uses the Responses API endpoint for advanced reasoning
- Standard agents use Chat Completions API

---

### Google Vertex AI (Gemini)

**Endpoint:** `{amp.url}/api/provider/google`  
**Source:** main.js:1295  
**Method:** POST  
**Proxied:** Yes (through amp.url)

**Models:**
- Gemini 2.0 Flash
- Gemini 1.5 Pro
- Gemini 1.5 Flash

**SDK Used:** Google Generative AI SDK (`@google/generative-ai`)

**Configuration:**
```typescript
const client = new GoogleGenerativeAI({
  baseURL: `${ampUrl}/api/provider/google`,
  apiKey: ampApiKey,
});
```

**Special Features:**
- Supports streaming responses (SSE)
- Native multimodal support (text, images, video)
- System instruction support

---

### xAI (Grok)

**Endpoint:** `{amp.url}/api/provider/xai/v1`  
**Source:** main.js:1939  
**Method:** POST  
**Proxied:** Yes (through amp.url)

**Models:**
- Grok Beta
- Grok 2

**SDK Used:** OpenAI-compatible SDK

**Configuration:**
```typescript
const client = new OpenAI({
  baseURL: `${ampUrl}/api/provider/xai/v1`,
  apiKey: ampApiKey,
});
```

**Special Features:**
- Supports streaming responses (SSE)
- OpenAI-compatible API structure
- Real-time information access

---

### Cerebras

**Endpoint:** `{amp.url}/api/provider/cerebras`  
**Source:** main.js:1930  
**Method:** POST  
**Proxied:** Yes (through amp.url)

**Models:**
- Llama 3.3 70B
- Llama 3.1 70B
- Qwen models

**SDK Used:** OpenAI-compatible SDK

**Configuration:**
```typescript
const client = new OpenAI({
  baseURL: `${ampUrl}/api/provider/cerebras`,
  apiKey: ampApiKey,
});
```

**Special Features:**
- Supports streaming responses (SSE)
- Ultra-fast inference speeds
- OpenAI-compatible API

---

### Groq

**Endpoint:** `{amp.url}/api/provider/groq`  
**Source:** main.js:1931  
**Method:** POST  
**Proxied:** Yes (through amp.url)

**Models:**
- Llama 3.1 models
- Mixtral models
- Gemma models

**SDK Used:** OpenAI-compatible SDK

**Configuration:**
```typescript
const client = new OpenAI({
  baseURL: `${ampUrl}/api/provider/groq`,
  apiKey: ampApiKey,
});
```

**Special Features:**
- Supports streaming responses (SSE)
- High-speed inference
- OpenAI-compatible API

---

### Supernova

**Endpoint:** `{amp.url}/api/provider/supernova/v1`  
**Source:** main.js:1939  
**Method:** POST  
**Proxied:** Yes (through amp.url)

**Models:**
- Supernova models (alternative xAI routing)

**SDK Used:** OpenAI-compatible SDK

**Configuration:**
```typescript
const client = new OpenAI({
  baseURL: `${ampUrl}/api/provider/supernova/v1`,
  apiKey: ampApiKey,
});
```

**Special Features:**
- Supports streaming responses (SSE)
- Alternative routing for xAI models
- OpenAI-compatible API

---

### OpenRouter (Direct Connection)

**Endpoint:** `https://openrouter.ai/api/v1`  
**Source:** main.js:1937  
**Method:** POST  
**Proxied:** ⚠️ **NO** - Direct connection to OpenRouter

**Models:**
- Kimi K2
- Sonoma Sky
- Various third-party models

**SDK Used:** OpenAI-compatible SDK

**Configuration:**
```typescript
const client = new OpenAI({
  baseURL: 'https://openrouter.ai/api/v1',
  apiKey: openRouterApiKey, // NOT amp.url API key
});
```

**Authentication Requirements:**
- Requires separate OpenRouter API key
- Must set `amp.openrouter.apiKey` or `OPENROUTER_API_KEY` environment variable
- **Does not use Amp API key**

**Special Features:**
- Supports streaming responses (SSE)
- Access to diverse model providers
- OpenAI-compatible API

**Important Notes:**
- This is the ONLY endpoint that bypasses the Amp proxy
- Requires separate OpenRouter account and API key
- Network policies must allow direct access to `openrouter.ai`
- Intermediary proxies must handle this differently from other providers

---

## User & Thread Management Endpoints

These endpoints manage user data and conversation threads. All are proxied through `amp.url`.

### Get Current User

**Endpoint:** `{amp.url}/api/user`  
**Source:** main.js:6460  
**Method:** GET  
**Proxied:** Yes

**Purpose:** Fetch current authenticated user information

**Usage:**
```typescript
GET /api/user
Authorization: Bearer {ampApiKey}
```

**Response:**
```json
{
  "id": "user-id",
  "email": "user@example.com",
  "name": "User Name"
}
```

---

### List User Threads

**Endpoint:** `{amp.url}/api/threads?createdByUserID={userId}`  
**Source:** main.js:6461  
**Method:** GET  
**Proxied:** Yes

**Purpose:** List all conversation threads created by the user

**Usage:**
```typescript
GET /api/threads?createdByUserID=user-id
Authorization: Bearer {ampApiKey}
```

**Response:**
```json
{
  "threads": [
    {
      "id": "thread-id",
      "title": "Thread title",
      "createdAt": "2025-01-21T00:00:00Z",
      "updatedAt": "2025-01-21T01:00:00Z"
    }
  ]
}
```

---

### Get Specific Thread

**Endpoint:** `{amp.url}/api/threads/{threadId}`  
**Source:** main.js:6457  
**Method:** GET  
**Proxied:** Yes

**Purpose:** Fetch a specific thread by ID (used before local cache lookup)

**Usage:**
```typescript
GET /api/threads/{threadId}
Authorization: Bearer {ampApiKey}
```

**Response:**
```json
{
  "id": "thread-id",
  "title": "Thread title",
  "messages": [...],
  "createdAt": "2025-01-21T00:00:00Z"
}
```

---

### Upload/Create Thread

**Endpoint:** `{amp.url}/api/threads` (via internal client method)  
**Source:** main.js:6451  
**Method:** POST  
**Proxied:** Yes

**Purpose:** Create or upload thread data to server

**Usage:**
```typescript
POST /api/threads
Authorization: Bearer {ampApiKey}
Content-Type: application/json

{
  "title": "Thread title",
  "messages": [...]
}
```

**Notes:**
- Called via internal client method `KJ.uploadThread`
- Synchronizes local thread state with server
- Used for thread persistence and sharing

---

## GitHub Integration Endpoints

The Librarian agent uses these endpoints to access GitHub repositories. All are proxied through `amp.url`.

### Check GitHub Authentication Status

**Endpoint:** `{amp.url}/api/internal/github-auth-status`  
**Source:** main.js:5151  
**Method:** GET  
**Proxied:** Yes

**Purpose:** Verify if user has authenticated with GitHub

**Usage:**
```typescript
GET /api/internal/github-auth-status
Authorization: Bearer {ampApiKey}
```

**Response:**
```json
{
  "authenticated": true,
  "scopes": ["repo", "user"]
}
```

**Notes:**
- Used by Librarian agent before accessing private repositories
- Returns authentication status and available OAuth scopes

---

### GitHub REST API Proxy

**Endpoint:** `{amp.url}/api/internal/github-proxy/{path}`  
**Source:** main.js:4859  
**Method:** ALL (GET, POST, etc.)  
**Proxied:** Yes

**Purpose:** Proxy GitHub REST API calls through Amp infrastructure

**Example Paths:**
- `/api/internal/github-proxy/repos/{owner}/{repo}/commits` - Get commits
- `/api/internal/github-proxy/repos/{owner}/{repo}/contents/{path}` - Get file contents
- `/api/internal/github-proxy/search/commits` - Search commits
- `/api/internal/github-proxy/repos/{owner}/{repo}` - Get repository info

**Usage:**
```typescript
GET /api/internal/github-proxy/repos/owner/repo/commits
Authorization: Bearer {ampApiKey}
```

**Notes:**
- Proxies all GitHub REST API v3 endpoints
- Handles authentication via user's GitHub OAuth token stored on Amp server
- Allows Librarian to access both public and private repositories (with permission)
- Path after `/github-proxy/` is forwarded directly to GitHub API

---

## Real-Time Connection Endpoints

### Server-Sent Events (SSE)

**Endpoints:** All provider inference endpoints support SSE when `stream: true` is set  
**Source:** main.js:1906, 1911-1912, 1936-1937  
**Method:** POST with `stream: true` parameter  
**Proxied:** Yes (except OpenRouter which is direct)

**Purpose:** Receive incremental model response chunks in real-time

**Event Types:**
- `message_start` - Response begins
- `content_block_start` - Content block begins
- `content_block_delta` - Incremental content
- `content_block_stop` - Content block ends
- `message_stop` - Response complete

**Usage:**
```typescript
POST /api/provider/anthropic
Authorization: Bearer {ampApiKey}
Content-Type: application/json

{
  "model": "claude-3-5-sonnet",
  "stream": true,
  "messages": [...]
}
```

**Response:** `text/event-stream` with incremental chunks

**Notes:**
- All proxied providers support SSE streaming
- OpenRouter also supports SSE but connects directly (not through amp.url)
- CLI uses SSE parsers to consume incremental deltas

---

### Amp Connect (Web Interface Sync)

**Endpoint:** `{amp.url}/api/connect`  
**Source:** Not explicitly found in bundle, but referenced in documentation  
**Method:** SSE  
**Proxied:** Yes

**Purpose:** Real-time synchronization between CLI and web interface

**Event Types:**
- `user-message` - User sent a message
- `approve-tool` - User approved a tool execution
- `reject-tool` - User rejected a tool execution
- `disconnect` - Connection closed

**Notes:**
- Enables bidirectional communication between CLI and web UI
- Used for remote approval/rejection of tool executions
- Maintains session state across interfaces

---

## Telemetry & Metrics Endpoints

**Status:** ⚠️ **NOT FOUND**

The Oracle analysis found **no explicit telemetry or metrics endpoints** in the bundled CLI code. Common patterns searched:
- `/api/otel/v1/metrics` (OpenTelemetry)
- `/api/telemetry`
- `/api/internal/telemetry`
- `/api/events`
- `/api/analytics`

**Notes:**
- The CLI may collect telemetry through other mechanisms not visible in the bundle
- Some usage data may be included in existing API calls (e.g., model inference requests)
- Check with Amp's privacy policy for details on data collection

---

## Summary Tables

### Model Provider Endpoints (Proxied)

| Provider | Endpoint Path | Source Line | Streaming | SDK |
|----------|--------------|-------------|-----------|-----|
| Anthropic | `/api/provider/anthropic` | 1225 | Yes | `@anthropic-ai/sdk` |
| OpenAI | `/api/provider/openai/v1` | 1930 | Yes | `openai` |
| Google Vertex | `/api/provider/google` | 1295 | Yes | `@google/generative-ai` |
| xAI | `/api/provider/xai/v1` | 1939 | Yes | OpenAI-compatible |
| Cerebras | `/api/provider/cerebras` | 1930 | Yes | OpenAI-compatible |
| Groq | `/api/provider/groq` | 1931 | Yes | OpenAI-compatible |
| Supernova | `/api/provider/supernova/v1` | 1939 | Yes | OpenAI-compatible |

**All proxied through:** `{amp.url}` (default: `https://ampcode.com`)

---

### Direct Connection Endpoints (NOT Proxied)

| Provider | Endpoint URL | Source Line | Streaming | Authentication |
|----------|-------------|-------------|-----------|----------------|
| OpenRouter | `https://openrouter.ai/api/v1` | 1937 | Yes | Separate API key required |

**Authentication:** `amp.openrouter.apiKey` or `OPENROUTER_API_KEY` environment variable

---

### Internal API Endpoints

| Endpoint | Method | Source Line | Purpose |
|----------|--------|-------------|---------|
| `/api/user` | GET | 6460 | Get current user info |
| `/api/threads` | GET | 6461 | List user's threads |
| `/api/threads/{id}` | GET | 6457 | Get specific thread |
| `/api/threads` | POST | 6451 | Upload/create thread |
| `/api/internal/github-auth-status` | GET | 5151 | Check GitHub auth |
| `/api/internal/github-proxy/{path}` | ALL | 4859 | Proxy GitHub API |

**All proxied through:** `{amp.url}`

---

## Request Headers

### Headers for Proxied Requests (through amp.url)

All requests to proxied endpoints should include:

```http
Authorization: Bearer {ampApiKey}
Content-Type: application/json
User-Agent: amp-cli/{version}
```

**Additional headers set by CLI:**
- `amp-application` - Application identifier
- `amp-message-id` - Request correlation ID (for debugging)
- `x-amp-stream` - Indicates streaming request

**Example:**
```http
POST https://ampcode.com/api/provider/anthropic
Authorization: Bearer sk-amp-xxxxx
Content-Type: application/json
User-Agent: amp-cli/0.0.1761153678
amp-application: cli
amp-message-id: msg_abc123

{
  "model": "claude-3-5-sonnet",
  "messages": [...]
}
```

---

### Headers for OpenRouter Requests (direct)

OpenRouter requests bypass Amp infrastructure and use OpenRouter's authentication:

```http
Authorization: Bearer {openRouterApiKey}
Content-Type: application/json
HTTP-Referer: https://ampcode.com
X-Title: Amp CLI
```

**Example:**
```http
POST https://openrouter.ai/api/v1/chat/completions
Authorization: Bearer sk-or-xxxxx
Content-Type: application/json
HTTP-Referer: https://ampcode.com
X-Title: Amp CLI

{
  "model": "kimi/kimi-k2",
  "messages": [...]
}
```

**Important:**
- Uses separate API key from `amp.openrouter.apiKey` or `OPENROUTER_API_KEY`
- **NOT** the same as `AMP_API_KEY`
- Requires active OpenRouter account

---

## Proxy Configuration Requirements

### For Network Administrators

If you need to configure an intermediary proxy for the AMP CLI, follow these guidelines:

#### 1. Proxied Traffic (through amp.url)

**Must proxy these patterns:**
```
{amp.url}/api/provider/*        # All model inference
{amp.url}/api/user              # User management
{amp.url}/api/threads*          # Thread management
{amp.url}/api/internal/*        # GitHub & services
{amp.url}/api/connect           # SSE connection
```

**Default amp.url:** `https://ampcode.com`

**Required Actions:**
- Forward all requests to configured `amp.url`
- Preserve `Authorization` header with Amp API key
- Preserve custom headers (`amp-application`, `amp-message-id`)
- Support `text/event-stream` content type for SSE
- Allow long-lived connections for streaming responses

---

#### 2. Direct Traffic (bypass proxy)

**Must allow direct access:**
```
https://openrouter.ai/api/v1/*   # OpenRouter direct
```

**Required Actions:**
- Allow direct HTTPS connection to `openrouter.ai`
- Do NOT modify authentication headers
- Support streaming responses

---

#### 3. Environment Configuration

Users can configure proxy via standard Node.js environment variables:

```bash
# HTTP/HTTPS proxy
export HTTP_PROXY=http://proxy.example.com:8080
export HTTPS_PROXY=http://proxy.example.com:8080

# No proxy exceptions (if needed)
export NO_PROXY=localhost,127.0.0.1

# Custom CA certificates (for corporate proxies)
export NODE_EXTRA_CA_CERTS=/path/to/ca-bundle.crt
```

**Amp-specific:**
```bash
# Override Amp server URL
export AMP_URL=https://custom-amp-server.com
```

---

#### 4. SSL/TLS Requirements

- All endpoints use HTTPS
- CLI validates SSL certificates by default
- Corporate proxies may need custom CA certificates (use `NODE_EXTRA_CA_CERTS`)
- Man-in-the-middle inspection must preserve valid certificates

---

#### 5. Firewall Rules

**Outbound HTTPS (443) required for:**
- `ampcode.com` (or custom amp.url domain)
- `openrouter.ai` (if using OpenRouter models)

**Ports:**
- 443 (HTTPS) - All endpoints
- No inbound ports required (CLI is client-only)

---

#### 6. Testing Proxy Configuration

Test proxy setup with these commands:

```bash
# Test Amp connectivity
amp login

# Test model inference (proxied)
echo "Hello" | amp

# Test OpenRouter (direct, if configured)
amp --model kimi/kimi-k2 "Hello"

# Debug connection issues
export AMP_LOG_LEVEL=debug
export AMP_LOG_FILE=amp-debug.log
amp "Test message"
cat amp-debug.log
```

---

## Notes

1. **Source line numbers** reference the minified bundle and are approximate anchors
2. **All `/api/provider/*` endpoints** are resolved against `amp.url` except OpenRouter
3. **OpenRouter is the ONLY direct connection** - all others are proxied
4. **Streaming support** (SSE) is available on all inference endpoints
5. **Authentication** uses Bearer token from `AMP_API_KEY` for proxied requests
6. **OpenRouter authentication** requires separate API key configuration

---

## Related Resources

- [Settings Reference](./settings.md) - Configuration settings including `amp.url`
- [Amp Manual](https://ampcode.com/manual) - Official documentation
- [API Configuration](https://ampcode.com/manual#configuration) - Configuration guide

---

**Generation Details:**
- CLI Version: 0.0.1761153678-gfa55cf
- Bundle: `node_modules/@sourcegraph/amp/dist/main.js`
- Analysis Method: Oracle-assisted extraction
- Verification: Cross-referenced with source code
- Last Updated: 2025-01-21
