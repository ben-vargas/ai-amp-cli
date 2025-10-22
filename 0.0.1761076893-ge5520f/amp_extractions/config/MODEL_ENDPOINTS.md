# Amp Model to Endpoint Routing Map

Complete mapping of which models route through which endpoints when proxied through `amp.url`.

All endpoints are constructed as: `new URL("/api/provider/{provider}", amp.url)`

---

## Provider Endpoint Mappings

### Anthropic (Claude Models)

**Endpoint**: `{amp.url}/api/provider/anthropic`

**Models**:
- `claude-sonnet-4-20250514` (Claude Sonnet 4)
- `claude-sonnet-4-5-20250929` (Claude Sonnet 4.5) - **Default for Smart mode**
- `claude-opus-4-20250514` (Claude Opus 4)
- `claude-opus-4-1-20250805` (Claude Opus 4.1)
- `claude-3-5-haiku-20241022` (Claude 3.5 Haiku)
- `claude-3-5-sonnet-20241022` (Claude 3.5 Sonnet)

**SDK Used**: Anthropic SDK
```javascript
new c4({
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

**SDK Used**: OpenAI SDK (using `openai` npm package)
```javascript
new V8({  // V8 = OpenAI class
  apiKey: apiKey,
  baseURL: new URL("/api/provider/openai/v1", ampURL).toString()
})
```

**Special Note**: Oracle tool uses OpenAI Responses API endpoint, not Chat Completions.

---

### Google Vertex AI (Gemini Models)

**Endpoint**: `{amp.url}/api/provider/google`

**Models**:
- `gemini-2.5-pro` (Gemini 2.5 Pro)
- `gemini-2.5-flash` (Gemini 2.5 Flash)
- `gemini-2.5-flash-preview-09-2025` (Gemini 2.5 Flash Preview)
- `gemini-2.5-flash-lite` (Gemini 2.5 Flash Lite)
- `gemini-2.5-flash-lite-preview-09-2025` (Gemini 2.5 Flash Lite Preview) - **Default for Free mode**

**SDK Used**: Google Generative AI SDK
```javascript
new SM1({  // SM1 = GoogleGenerativeAI class
  apiKey: "placeholder",
  vertexai: true,
  googleAuthOptions: {},
  httpOptions: {
    baseUrl: new URL("/api/provider/google", ampURL).toString(),
    headers: { Authorization: "Bearer " + apiKey, ... }
  }
})
```

---

### xAI (Grok Models)

**Endpoint**: `{amp.url}/api/provider/xai/v1`

**Models**:
- `grok-code-fast-1` (Grok Code Fast 1) - **Default for Fast mode**
- `grok-code` (Grok Code) - **Used in Free mode**

**SDK Used**: OpenAI SDK (xAI API is OpenAI-compatible)
```javascript
new V8({
  apiKey: apiKey,
  baseURL: new URL("/api/provider/xai/v1", ampURL).toString()
})
```

---

### Cerebras (Qwen Models)

**Endpoint**: `{amp.url}/api/provider/cerebras`

**Models**:
- `qwen-3-coder-480b` (Qwen 3 Coder 480B)

**SDK Used**: Custom Cerebras SDK (`qM0`)
```javascript
new qM0({
  apiKey: apiKey,
  baseURL: new URL("/api/provider/cerebras", ampURL).toString()
})
```

---

### Groq (OpenAI-compatible Models)

**Endpoint**: `{amp.url}/api/provider/groq`

**Models**:
- `openai/gpt-oss-120b` (GPT OSS 120B)

**SDK Used**: OpenAI SDK (Groq API is OpenAI-compatible)
```javascript
new V8({
  apiKey: apiKey,
  baseURL: new URL("/api/provider/groq", ampURL).toString()
})
```

---

### Moonshot AI (via OpenRouter)

**Endpoint**: `https://openrouter.ai/api/v1` (NOT proxied through amp.url)

**Models**:
- `kimi-k2-instruct-0905` (Kimi K2 Instruct)

**SDK Used**: OpenAI SDK
```javascript
new V8({
  apiKey: config.settings["openrouter.apiKey"] || process.env.OPENROUTER_API_KEY,
  baseURL: "https://openrouter.ai/api/v1"
})
```

**Note**: When using model `"kimi-k2-instruct-0905"`, it's sent as `"moonshotai/kimi-k2-instruct-0905"` to OpenRouter.

---

### OpenRouter (Sonoma Sky Alpha)

**Endpoint**: `https://openrouter.ai/api/v1` (NOT proxied through amp.url)

**Models**:
- `sonoma-sky-alpha` (Sonoma Sky Alpha)

**SDK Used**: OpenAI SDK
```javascript
new V8({
  apiKey: config.settings["openrouter.apiKey"] || process.env.OPENROUTER_API_KEY,
  baseURL: "https://openrouter.ai/api/v1"
})
```

**Note**: Model is sent as `"openrouter/sonoma-sky-alpha"` to the API.

---

### Supernova (Code Models)

**Endpoint**: `{amp.url}/api/provider/supernova/v1`

**Models**:
- `code-supernova` (Code Supernova)

**SDK Used**: OpenAI SDK (Supernova API is OpenAI-compatible)
```javascript
new V8({
  apiKey: apiKey,
  baseURL: new URL("/api/provider/supernova/v1", ampURL).toString()
})
```

---

## Summary Table

| Provider | Models Count | Routes Through amp.url? | Base Path |
|----------|--------------|-------------------------|-----------|
| Anthropic | 6 | ✅ Yes | `/api/provider/anthropic` |
| OpenAI | 13 | ✅ Yes | `/api/provider/openai/v1` |
| Google Vertex AI | 5 | ✅ Yes | `/api/provider/google` |
| xAI | 2 | ✅ Yes | `/api/provider/xai/v1` |
| Cerebras | 1 | ✅ Yes | `/api/provider/cerebras` |
| Groq | 1 | ✅ Yes | `/api/provider/groq` |
| Supernova | 1 | ✅ Yes | `/api/provider/supernova/v1` |
| **OpenRouter** | **2** | **❌ No** | `https://openrouter.ai/api/v1` |

---

## Special Cases

### Oracle Tool
- **Model**: Determined by `amp.internal.oracleModel` (defaults to `"gpt-5"`)
- **Endpoint**: Same as OpenAI models → `{amp.url}/api/provider/openai/v1`
- **API**: Uses OpenAI **Responses API** (`/v1/responses`), not Chat Completions
- **Reasoning**: Enabled with `reasoning.effort` = `"medium"` or `"high"`

### OpenRouter Models
These are the **ONLY** models that bypass `amp.url` entirely:
1. `sonoma-sky-alpha` (used in `sonomoon:*` agent modes)
2. `kimi-k2-instruct-0905` (used in `kashmir:*` agent modes)

**Requirement**: Must set `amp.openrouter.apiKey` or `OPENROUTER_API_KEY` environment variable.

---

## Agent Mode to Endpoint Mapping

| Agent Mode | Primary Model | Endpoint |
|------------|---------------|----------|
| `smart` | Claude Sonnet 4.5 | `/api/provider/anthropic` |
| `fast` | Grok Code Fast 1 | `/api/provider/xai/v1` |
| `free` | Grok Code | `/api/provider/xai/v1` |
| `geppetto:main` | GPT-5 | `/api/provider/openai/v1` |
| `pinocchio:main` | GPT-5 Codex | `/api/provider/openai/v1` |
| `claudius:main` | Claude Opus 4 | `/api/provider/anthropic` |
| `opus4.1` | Claude Opus 4.1 | `/api/provider/anthropic` |
| `bolt:main` | Qwen 3 Coder 480B | `/api/provider/cerebras` |
| `hydrogen:main` | Code Supernova | `/api/provider/supernova/v1` |
| `kashmir:main` | Kimi K2 | `https://openrouter.ai/api/v1` ⚠️ |
| `gossip:main` | GPT OSS 120B | `/api/provider/groq` |
| `sonomoon:main` | Sonoma Sky Alpha | `https://openrouter.ai/api/v1` ⚠️ |

**⚠️** = Requires OpenRouter API key, not proxied through amp.url

---

## Request Headers

All proxied requests (through amp.url) include:
```javascript
headers: {
  "Authorization": "Bearer " + apiKey,
  "amp-application": "amp.chat",
  "amp-message-id": messageId,       // when available
  "amp-thread-id": threadId,         // when available
  // ... plus workspace/tree metadata
}
```

OpenRouter requests include:
```javascript
headers: {
  "amp-application": "amp.chat",
  "HTTP-Referer": threadID,
  "x-grok-conv-id": threadID         // for xAI models
}
```

---

## Authentication Flow

1. **Get API Key**: Retrieved from `secrets.getToken("apiKey", amp.url)`
2. **Construct Client**: Provider-specific SDK with baseURL
3. **Make Request**: SDK handles the actual HTTP request to the proxy
4. **Proxy Routes**: Amp server at `amp.url` proxies to actual provider API

**Exception**: OpenRouter makes direct HTTPS connections to `openrouter.ai`, not through Amp servers.

---

## Configuration Examples

### Use GPT-5 Mini for Oracle
```json
{
  "amp.internal.oracleModel": "gpt-5-mini"
}
```

### Switch to Geppetto Mode (GPT-5 primary)
```json
{
  "amp.experimental.agentMode": "geppetto:main"
}
```

### Use OpenRouter Models
```json
{
  "amp.openrouter.apiKey": "sk-or-v1-...",
  "amp.experimental.agentMode": "sonomoon:main"
}
```

### Use Vertex AI for Claude
```json
{
  "amp.anthropic.provider": "vertex"
}
```

---

**Generated from**: Amp CLI source code analysis (v0.0.1760587291-gab1037)
