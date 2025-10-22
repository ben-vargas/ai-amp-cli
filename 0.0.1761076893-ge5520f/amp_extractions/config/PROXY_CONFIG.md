# Configuring ai-cli-proxy-api to Work with Amp CLI

Complete guide to route Amp CLI requests through ai-cli-proxy-api to use alternative AI providers.

---

## Overview

Amp CLI expects routes like:
```
{amp.url}/api/provider/{provider}/v1/...
```

ai-cli-proxy-api currently provides:
```
http://localhost:8317/v1/...
```

**Solution**: Modify ai-cli-proxy-api to add Amp-compatible route aliases.

---

## Quick Start (TL;DR)

1. Clone and navigate:
   ```bash
   cd /home/code/projects/ben-vargas/ai-cli-proxy-api/main
   ```

2. Edit `internal/api/server.go` and add the code block shown below after line 293

3. Rebuild:
   ```bash
   go build -o cli-proxy-api ./cmd/server
   ```

4. Configure Amp:
   ```bash
   echo '{"amp.url": "http://localhost:8317"}' > ~/.config/amp/settings.json
   ```

5. Start proxy and use Amp

---

## Detailed Implementation

### Step 1: Modify the Route Handler

**File**: `internal/api/server.go`  
**Function**: `setupRoutes()` (starts at line 266)  
**Location**: After the `v1beta` group definition (after line 293)

Add this code block:

```go
// Amp CLI provider-style route aliases
// These routes allow Amp CLI to use paths like /api/provider/{provider}/v1/...
ampProviders := s.engine.Group("/api/provider")
ampProviders.Use(AuthMiddleware(s.accessManager))
{
	// Dynamic provider routing - accepts any provider name
	// Provider names: openai, anthropic, google, xai, groq, cerebras, supernova
	provider := ampProviders.Group("/:provider")
	
	// v1 endpoints (OpenAI + Claude compatible)
	v1Amp := provider.Group("/v1")
	{
		// Provider-aware models endpoint
		// Returns appropriate model list based on provider segment
		v1Amp.GET("/models", func(c *gin.Context) {
			providerName := strings.ToLower(c.Param("provider"))
			switch providerName {
			case "anthropic":
				// Return Claude models for Anthropic provider
				claudeCodeHandlers.ClaudeModels(c)
			case "google":
				// Return Gemini models for Google provider (via OpenAI-compatible interface)
				geminiHandlers.GeminiModels(c)
			default:
				// openai, xai, groq, cerebras, supernova use OpenAI-compatible models
				openaiHandlers.OpenAIModels(c)
			}
		})
		
		// OpenAI-compatible endpoints
		// Used by: openai, xai, groq, cerebras, supernova providers
		v1Amp.POST("/chat/completions", openaiHandlers.ChatCompletions)
		v1Amp.POST("/completions", openaiHandlers.Completions)
		v1Amp.POST("/responses", openaiResponsesHandlers.Responses)
		
		// Claude/Anthropic-compatible endpoints
		// Used by: anthropic provider
		v1Amp.POST("/messages", claudeCodeHandlers.ClaudeMessages)
		v1Amp.POST("/messages/count_tokens", claudeCodeHandlers.ClaudeCountTokens)
	}
	
	// v1beta endpoints (Gemini compatible)
	// Used by: google provider for native Gemini API calls
	v1betaAmp := provider.Group("/v1beta")
	{
		v1betaAmp.GET("/models", geminiHandlers.GeminiModels)
		v1betaAmp.POST("/models/:action", geminiHandlers.GeminiHandler)
		v1betaAmp.GET("/models/:action", geminiHandlers.GeminiGetHandler)
	}
}
```

**Full context of where to insert**:

```go
func (s *Server) setupRoutes() {
	s.engine.GET("/management.html", s.serveManagementControlPanel)
	openaiHandlers := openai.NewOpenAIAPIHandler(s.handlers)
	geminiHandlers := gemini.NewGeminiAPIHandler(s.handlers)
	geminiCLIHandlers := gemini.NewGeminiCLIAPIHandler(s.handlers)
	claudeCodeHandlers := claude.NewClaudeCodeAPIHandler(s.handlers)
	openaiResponsesHandlers := openai.NewOpenAIResponsesAPIHandler(s.handlers)

	// OpenAI compatible API routes
	v1 := s.engine.Group("/v1")
	v1.Use(AuthMiddleware(s.accessManager))
	{
		v1.GET("/models", s.unifiedModelsHandler(openaiHandlers, claudeCodeHandlers))
		v1.POST("/chat/completions", openaiHandlers.ChatCompletions)
		v1.POST("/completions", openaiHandlers.Completions)
		v1.POST("/messages", claudeCodeHandlers.ClaudeMessages)
		v1.POST("/messages/count_tokens", claudeCodeHandlers.ClaudeCountTokens)
		v1.POST("/responses", openaiResponsesHandlers.Responses)
	}

	// Gemini compatible API routes
	v1beta := s.engine.Group("/v1beta")
	v1beta.Use(AuthMiddleware(s.accessManager))
	{
		v1beta.GET("/models", geminiHandlers.GeminiModels)
		v1beta.POST("/models/:action", geminiHandlers.GeminiHandler)
		v1beta.GET("/models/:action", geminiHandlers.GeminiGetHandler)
	}

	// ========================================
	// INSERT THE AMP PROVIDER BLOCK HERE
	// ========================================

	// Root endpoint
	s.engine.GET("/", func(c *gin.Context) {
		// ... existing code continues
```

### Step 2: Rebuild the Binary

```bash
cd /home/code/projects/ben-vargas/ai-cli-proxy-api/main
go build -o cli-proxy-api ./cmd/server
```

If you get dependency errors:
```bash
go mod tidy
go build -o cli-proxy-api ./cmd/server
```

### Step 3: Configure ai-cli-proxy-api

Edit `config.yaml` (or use `config.example.yaml` as a template):

```yaml
# Server port
port: 8317

# Authentication
auth-dir: "~/.cli-proxy-api"
api-keys:
  - "your-api-key-1"  # This is what Amp will use to authenticate

# Provider configurations
# Configure the providers you want to use

# For Gemini/Google
generative-language-api-key:
  - "AIzaSy...your-google-api-key"

# For OpenAI Codex
codex-api-key:
  - api-key: "sk-...your-openai-key"

# For Claude
claude-api-key:
  - api-key: "sk-ant-...your-anthropic-key"

# Or use OpenAI-compatible upstream providers
openai-compatibility:
  - name: "openrouter"
    base-url: "https://openrouter.ai/api/v1"
    api-key-entries:
      - api-key: "sk-or-v1-...your-openrouter-key"
    models:
      - name: "anthropic/claude-3.5-sonnet"
        alias: "claude-3-5-sonnet-20241022"
      - name: "google/gemini-2.0-flash-exp:free"
        alias: "gemini-2.5-flash-lite"

# Proxy settings (optional)
proxy-url: ""  # Leave empty unless you need upstream proxy
debug: true    # Enable for troubleshooting
```

### Step 4: Start ai-cli-proxy-api

```bash
cd /home/code/projects/ben-vargas/ai-cli-proxy-api/main
./cli-proxy-api
```

Or if you need to specify a config file:
```bash
./cli-proxy-api --config /path/to/config.yaml
```

### Step 5: Configure Amp CLI

Edit `~/.config/amp/settings.json`:

```json
{
  "amp.url": "http://localhost:8317",
  "amp.git.commit.coauthor.enabled": false
}
```

### Step 6: Configure Secrets

Create/edit `~/.local/share/amp/secrets.json`:

```json
{
  "apiKey@http://localhost:8317": "your-api-key-1"
}
```

**Note**: The secret key format is `{secretName}@{ampUrl}`, and the value must match one of the `api-keys` in your ai-cli-proxy-api `config.yaml`.

---

## Testing the Setup

### Test 1: Verify Proxy Routes

```bash
# Test OpenAI-style endpoint
curl -H "Authorization: Bearer your-api-key-1" \
     -H "Content-Type: application/json" \
     -d '{"model":"gpt-4","messages":[{"role":"user","content":"hi"}]}' \
     http://localhost:8317/api/provider/openai/v1/chat/completions

# Test Claude endpoint
curl -H "Authorization: Bearer your-api-key-1" \
     -H "Content-Type: application/json" \
     -d '{"model":"claude-3-5-sonnet-20241022","messages":[{"role":"user","content":"hi"}],"max_tokens":100}' \
     http://localhost:8317/api/provider/anthropic/v1/messages

# Test models listing
curl -H "Authorization: Bearer your-api-key-1" \
     http://localhost:8317/api/provider/openai/v1/models
```

### Test 2: Test with Amp CLI

```bash
# Simple test
echo "what is 2+2?" | amp -x

# Check logs
tail -f /home/code/projects/ben-vargas/ai-cli-proxy-api/main/logs/*.log
```

---

## Provider Routing Map

When Amp makes requests, here's how they route through your proxy:

| Amp Provider | Amp Calls | Proxy Routes To | Upstream |
|--------------|-----------|-----------------|----------|
| `openai` | `/api/provider/openai/v1/chat/completions` | `/v1/chat/completions` | Your configured OpenAI key or upstream |
| `anthropic` | `/api/provider/anthropic/v1/messages` | `/v1/messages` | Your configured Claude key or upstream |
| `google` | `/api/provider/google/v1/chat/completions` | `/v1/chat/completions` | Gemini via OpenAI-compatible interface |
| `xai` | `/api/provider/xai/v1/chat/completions` | `/v1/chat/completions` | Your Grok auth or upstream |
| `cerebras` | `/api/provider/cerebras/v1/chat/completions` | `/v1/chat/completions` | Your Cerebras auth or upstream |
| `groq` | `/api/provider/groq/v1/chat/completions` | `/v1/chat/completions` | Your Groq auth or upstream |
| `supernova` | `/api/provider/supernova/v1/chat/completions` | `/v1/chat/completions` | Your Supernova auth or upstream |

**Key insight**: The `:provider` segment in the route is ignored - all providers route to the same underlying handlers. The proxy determines the actual upstream based on the `model` parameter in the request body.

---

## Model Mapping

Configure model aliases in `config.yaml` to map Amp's expected model names to your provider's actual model names:

### Example: Route Amp's Claude requests to OpenRouter

```yaml
openai-compatibility:
  - name: "openrouter"
    base-url: "https://openrouter.ai/api/v1"
    api-key-entries:
      - api-key: "sk-or-v1-...your-key"
    models:
      # Map Amp's Claude model names to OpenRouter
      - name: "anthropic/claude-sonnet-4.5-20250929"
        alias: "claude-sonnet-4-5-20250929"
      - name: "anthropic/claude-opus-4-20250514"
        alias: "claude-opus-4-20250514"
      - name: "anthropic/claude-3-5-haiku-20241022"
        alias: "claude-3-5-haiku-20241022"
      
      # Map Amp's GPT models
      - name: "openai/gpt-4o"
        alias: "gpt-4o"
      
      # Map Amp's Gemini models
      - name: "google/gemini-2.0-flash-exp:free"
        alias: "gemini-2.5-flash-lite"
```

### Example: Use Native Provider Credentials

```yaml
# Use your own Anthropic API key directly
claude-api-key:
  - api-key: "sk-ant-...your-key"

# Use your own Google API key
generative-language-api-key:
  - "AIzaSy...your-key"

# Use your own OpenAI key
codex-api-key:
  - api-key: "sk-...your-key"
```

---

## Authentication Flow

```
Amp CLI Request
    ↓ (sends Authorization: Bearer your-api-key-1)
http://localhost:8317/api/provider/openai/v1/chat/completions
    ↓ (ai-cli-proxy-api AuthMiddleware validates)
    ↓ (routes to ChatCompletions handler)
    ↓ (proxy adds upstream auth for OpenAI/Claude/etc)
Upstream Provider API (OpenAI, Claude, OpenRouter, etc.)
```

**Two authentication layers**:
1. **Amp → Proxy**: Uses `api-keys` from `config.yaml` (matched with Amp's secrets)
2. **Proxy → Upstream**: Uses provider-specific keys (claude-api-key, codex-api-key, etc.)

---

## Advanced Configuration

### Load Balancing Multiple Accounts

ai-cli-proxy-api supports multiple accounts with round-robin load balancing:

```yaml
# Multiple Claude accounts
claude-api-key:
  - api-key: "sk-ant-key1"
  - api-key: "sk-ant-key2"
  - api-key: "sk-ant-key3"

# Or use OAuth for Claude Code
# Run: ./cli-proxy-api --claude-login
# Then accounts are stored in ~/.cli-proxy-api/
```

### Per-Provider Proxy Settings

```yaml
# Global proxy for all requests
proxy-url: "socks5://127.0.0.1:1080"

# Or per-key proxy overrides
codex-api-key:
  - api-key: "sk-key1"
    proxy-url: "socks5://proxy1.example.com:1080"
  - api-key: "sk-key2"
    proxy-url: "http://proxy2.example.com:8080"
```

### Request Retry Configuration

```yaml
# Retry on transient errors
request-retry: 3

# Quota handling
quota-exceeded:
  switch-project: true
  switch-preview-model: true
```

---

## Usage Examples

### Example 1: Use Free Gemini via Google API

```yaml
# config.yaml
port: 8317
auth-dir: "~/.cli-proxy-api"
api-keys:
  - "amp-proxy-secret-123"

generative-language-api-key:
  - "AIzaSy...your-google-api-key"
```

```bash
# Amp secrets
echo '{"apiKey@http://localhost:8317": "amp-proxy-secret-123"}' > ~/.local/share/amp/secrets.json

# Amp settings
echo '{"amp.url": "http://localhost:8317"}' > ~/.config/amp/settings.json

# Use Amp normally - it will route through proxy to Gemini
amp
```

### Example 2: Route All Providers Through OpenRouter

```yaml
# config.yaml
openai-compatibility:
  - name: "openrouter"
    base-url: "https://openrouter.ai/api/v1"
    api-key-entries:
      - api-key: "sk-or-v1-...key"
    models:
      # Claude models
      - name: "anthropic/claude-sonnet-4.5-20250929"
        alias: "claude-sonnet-4-5-20250929"
      - name: "anthropic/claude-opus-4-1-20250805"
        alias: "claude-opus-4-1-20250805"
      
      # OpenAI models
      - name: "openai/gpt-4o"
        alias: "gpt-4o"
      - name: "openai/gpt-5"
        alias: "gpt-5"  # If OpenRouter supports it
      
      # Gemini models
      - name: "google/gemini-2.0-flash-exp:free"
        alias: "gemini-2.5-flash-lite"
      - name: "google/gemini-2.0-flash-thinking-exp:free"
        alias: "gemini-2.5-pro"
      
      # XAI models
      - name: "x-ai/grok-2-1212"
        alias: "grok-code-fast-1"
```

### Example 3: Mix Native Credentials with OpenRouter

```yaml
# Use native Claude API key
claude-api-key:
  - api-key: "sk-ant-...your-claude-key"

# Route OpenAI through OpenRouter
openai-compatibility:
  - name: "openrouter"
    base-url: "https://openrouter.ai/api/v1"
    api-key-entries:
      - api-key: "sk-or-v1-..."
    models:
      - name: "openai/gpt-4o"
        alias: "gpt-4o"

# Use Google native for Gemini
generative-language-api-key:
  - "AIzaSy...your-key"
```

---

## Troubleshooting

### Issue: 404 Not Found

**Symptom**: Amp shows "404" errors

**Check**:
```bash
# Verify routes are registered
curl http://localhost:8317/api/provider/openai/v1/models

# Check proxy logs
tail -f logs/*.log
```

**Fix**: Ensure you rebuilt after editing server.go

### Issue: 401 Unauthorized

**Symptom**: Authentication failures

**Check**:
```bash
# Verify secret format
cat ~/.local/share/amp/secrets.json
# Should be: {"apiKey@http://localhost:8317": "your-api-key-1"}

# Verify proxy config
cat config.yaml | grep -A5 api-keys
```

**Fix**: The key in Amp secrets must match an entry in proxy's `api-keys` array

### Issue: Model Not Found

**Symptom**: Amp complains about unsupported model

**Solutions**:

1. **Check model mapping**: Verify alias in `config.yaml` matches Amp's expected name
2. **Check provider has model**: `curl -H "Authorization: Bearer key" http://localhost:8317/v1/models`
3. **Enable debug logging**: Set `debug: true` in `config.yaml`

### Issue: Context Window Too Small

Some free models have smaller context windows than Amp expects.

**Fix**: Use model aliases to map to models with larger context:
```yaml
models:
  - name: "google/gemini-2.0-flash-thinking-exp-1219:free"
    alias: "gemini-2.5-pro"  # Amp expects this for large context
```

---

## Alternative: Reverse Proxy (No Code Changes)

If you prefer not to modify ai-cli-proxy-api source:

### Option A: Nginx

```nginx
server {
    listen 8318;
    
    # Rewrite Amp paths to proxy paths
    location ~ ^/api/provider/[^/]+/v1/(.+)$ {
        rewrite ^/api/provider/[^/]+/v1/(.+)$ /v1/$1 break;
        proxy_pass http://127.0.0.1:8317;
        proxy_set_header Host $host;
        proxy_set_header Authorization $http_authorization;
    }
    
    location ~ ^/api/provider/[^/]+/v1beta/(.+)$ {
        rewrite ^/api/provider/[^/]+/v1beta/(.+)$ /v1beta/$1 break;
        proxy_pass http://127.0.0.1:8317;
        proxy_set_header Host $host;
        proxy_set_header Authorization $http_authorization;
    }
}
```

Then set `amp.url=http://localhost:8318`

### Option B: Caddy

```
:8318 {
    @ampProvider path_regexp provider ^/api/provider/[^/]+/v1/(.+)$
    handle @ampProvider {
        uri path_regexp ^/api/provider/[^/]+/v1/(.+)$ /v1/$1
        reverse_proxy 127.0.0.1:8317
    }
    
    @ampProviderBeta path_regexp providerBeta ^/api/provider/[^/]+/v1beta/(.+)$
    handle @ampProviderBeta {
        uri path_regexp ^/api/provider/[^/]+/v1beta/(.+)$ /v1beta/$1
        reverse_proxy 127.0.0.1:8317
    }
}
```

---

## Verifying Everything Works

### Complete Test Flow

```bash
# 1. Start the proxy
cd /home/code/projects/ben-vargas/ai-cli-proxy-api/main
./cli-proxy-api

# 2. In another terminal, test proxy directly
curl -H "Authorization: Bearer your-api-key-1" \
     -H "Content-Type: application/json" \
     -d '{"model":"gemini-2.5-pro","messages":[{"role":"user","content":"Say hi"}]}' \
     http://localhost:8317/api/provider/google/v1/chat/completions

# 3. Test with Amp CLI
echo "what is today's date?" | amp -x

# 4. Check logs
tail -f logs/*.log
# Should show requests coming in with model names and provider routing
```

---

## Provider-Specific Notes

### Google/Gemini
- Amp calls `/v1/chat/completions` (OpenAI-compatible interface)
- Proxy translates to Gemini API calls
- Requires: `generative-language-api-key` or OAuth login

### Anthropic/Claude
- Amp calls `/v1/messages` (Claude native API)
- Direct passthrough to Claude API
- Requires: `claude-api-key` or OAuth login (`--claude-login`)

### OpenAI/GPT
- Amp calls `/v1/chat/completions` or `/v1/responses` (for reasoning models)
- Requires: `codex-api-key` or OAuth login (`--codex-login`)

### XAI/Grok
- Amp calls `/v1/chat/completions`
- Route through OpenRouter or native Grok API if supported
- **Tip**: Use OpenRouter with model `x-ai/grok-2-1212`

### Others (Cerebras, Groq, Supernova)
- All use OpenAI-compatible interface
- Route through openai-compatibility config or native keys

---

## Model Name Translation Table

Map Amp's expected model names to your upstream provider:

| Amp Model Name | Suggested Alias/Upstream | Provider |
|----------------|--------------------------|----------|
| `claude-sonnet-4-5-20250929` | `anthropic/claude-sonnet-4.5-20250929` | OpenRouter or Anthropic |
| `claude-opus-4-1-20250805` | `anthropic/claude-opus-4-1-20250805` | OpenRouter or Anthropic |
| `gpt-5` | `openai/gpt-5` | OpenRouter (if available) |
| `gpt-4o` | `gpt-4o` | OpenAI or OpenRouter |
| `gemini-2.5-pro` | `gemini-2.0-flash-thinking-exp` | Google Gemini API |
| `gemini-2.5-flash-lite` | `gemini-2.0-flash-exp:free` | Google Gemini API |
| `grok-code-fast-1` | `x-ai/grok-2-1212` | OpenRouter |
| `grok-code` | `x-ai/grok-beta` | OpenRouter |

---

## Full Example Configuration

### ai-cli-proxy-api config.yaml

```yaml
port: 8317
auth-dir: "~/.cli-proxy-api"
api-keys:
  - "amp-local-proxy-key"

debug: true
logging-to-file: true

# Route everything through OpenRouter
openai-compatibility:
  - name: "openrouter"
    base-url: "https://openrouter.ai/api/v1"
    api-key-entries:
      - api-key: "sk-or-v1-YOUR-OPENROUTER-KEY"
    models:
      # Claude models (for Amp's anthropic provider)
      - name: "anthropic/claude-sonnet-4.5-20250929"
        alias: "claude-sonnet-4-5-20250929"
      - name: "anthropic/claude-3.5-sonnet-20241022"
        alias: "claude-3-5-sonnet-20241022"
      
      # OpenAI models (for Amp's openai provider)
      - name: "openai/gpt-4o"
        alias: "gpt-4o"
      
      # Gemini models (for Amp's google provider)
      - name: "google/gemini-2.0-flash-thinking-exp-1219:free"
        alias: "gemini-2.5-pro"
      - name: "google/gemini-2.0-flash-exp:free"
        alias: "gemini-2.5-flash-lite"
      
      # XAI models (for Amp's xai provider)
      - name: "x-ai/grok-2-1212"
        alias: "grok-code-fast-1"
      - name: "x-ai/grok-beta"
        alias: "grok-code"
      
      # Deepseek (if using groq or other providers)
      - name: "deepseek/deepseek-chat"
        alias: "qwen-3-coder-480b"
```

### Amp Settings (~/.config/amp/settings.json)

```json
{
  "amp.url": "http://localhost:8317",
  "amp.git.commit.coauthor.enabled": false,
  "amp.experimental.agentMode": "smart"
}
```

### Amp Secrets (~/.local/share/amp/secrets.json)

```json
{
  "apiKey@http://localhost:8317": "amp-local-proxy-key"
}
```

---

## Benefits of This Setup

✅ **Cost savings**: Use free tier models via OpenRouter  
✅ **Multi-provider**: Route different Amp models to different upstreams  
✅ **Load balancing**: Multiple accounts automatically balanced  
✅ **Centralized auth**: Manage all provider keys in one place  
✅ **Offline dev**: Use local models or custom endpoints  
✅ **Privacy**: Keep conversations local if using local LLMs  

---

## Common Use Cases

### Use Case 1: Free Tier Only

Route all Amp requests to free OpenRouter models:

```yaml
openai-compatibility:
  - name: "openrouter-free"
    base-url: "https://openrouter.ai/api/v1"
    api-key-entries:
      - api-key: "sk-or-v1-..."
    models:
      - name: "google/gemini-2.0-flash-exp:free"
        alias: "gemini-2.5-flash-lite"
      - name: "anthropic/claude-3.5-sonnet:free"
        alias: "claude-sonnet-4-5-20250929"
```

### Use Case 2: Local LLM (Ollama/LM Studio)

```yaml
openai-compatibility:
  - name: "local-llama"
    base-url: "http://localhost:11434/v1"  # Ollama
    api-keys:
      - "dummy"
    models:
      - name: "llama3.3"
        alias: "gpt-4o"  # Amp will think it's GPT-4o
```

### Use Case 3: Mix of Native + OpenRouter

```yaml
# Native Claude (paid tier, fast)
claude-api-key:
  - api-key: "sk-ant-..."

# Free Gemini via Google
generative-language-api-key:
  - "AIzaSy..."

# Everything else via OpenRouter
openai-compatibility:
  - name: "openrouter"
    base-url: "https://openrouter.ai/api/v1"
    api-key-entries:
      - api-key: "sk-or-v1-..."
    models:
      - name: "openai/gpt-4o"
        alias: "gpt-4o"
      - name: "x-ai/grok-2-1212"
        alias: "grok-code-fast-1"
```

---

## Security Considerations

1. **Localhost only**: By default, proxy binds to `0.0.0.0:8317`. For local-only access:
   ```yaml
   # Not directly configurable in current version
   # Use firewall rules or bind to 127.0.0.1 via reverse proxy
   ```

2. **API key rotation**: Change keys in `config.yaml`, proxy auto-reloads

3. **Upstream key security**: Keep your `config.yaml` secure with `chmod 600`

4. **Separate keys**: Use different `api-keys` entry for Amp vs other clients

---

## Monitoring and Debugging

### Enable Debug Logs

```yaml
debug: true
logging-to-file: true
```

Logs location: `logs/*.log` in proxy directory

### Watch Requests in Real-Time

```bash
tail -f /home/code/projects/ben-vargas/ai-cli-proxy-api/main/logs/*.log
```

### Check Usage Statistics

If you enabled the management API:

```bash
# Get usage stats
curl -H "Authorization: Bearer your-management-key" \
     http://localhost:8317/v0/management/usage
```

---

## Next Steps

1. ✅ Modify `internal/api/server.go` per instructions above
2. ✅ Rebuild binary: `go build -o cli-proxy-api ./cmd/server`
3. ✅ Configure `config.yaml` with your provider keys
4. ✅ Set Amp secrets and settings as shown
5. ✅ Test with `echo "test" | amp -x`
6. ✅ Monitor logs to verify routing

---

## Oracle Review Summary

✅ **Approach validated**: Route aliasing is the cleanest solution  
✅ **Authentication works**: Middleware chain is correct  
✅ **Provider routing**: Model-driven via registry, properly configured  
⚠️ **Models endpoint improved**: Now returns provider-specific model lists  
✅ **No major gotchas**: Implementation aligns with both codebases  

**Alternative considered**: Single rewrite handler instead of route duplication - also valid but slightly trickier for middleware ordering.

---

**Generated**: Analysis of ai-cli-proxy-api source code structure  
**Last Updated**: Based on current codebase at /home/code/projects/ben-vargas/ai-cli-proxy-api/main
