# AMP Agent Tools Documentation

**Source:** AMP CLI `main.js` (version 0.0.1761153678-gfa55cf)  
**Extracted:** Complete tool definitions and implementations for all agents

---

## Table of Contents

1. [Librarian Agent Tools](#librarian-agent-tools)
2. [Oracle Agent Tools](#oracle-agent-tools)
3. [Smart Agent Tools](#smart-agent-tools)
4. [Tool Implementation Patterns](#tool-implementation-patterns)

---

## Librarian Agent Tools

The Librarian agent has access to 7 specialized GitHub tools for multi-repository codebase understanding.

**Tool Array Variable:** `Xw1` (Line 5149)  
**Tools:** `[LO0, AO0, FO0, BO0, VO0, NO0, MO0]`

### 1. Read (`read` / LO0)

**Variable:** `LO0` (Lines 5045-5078)  
**Function:** `rM8` (Lines 5044-5078)

#### Description

Read the contents of a file from a GitHub repository.

#### When to Use
- When you need to examine the contents of a specific file
- When you want to understand implementation details
- When you need to see the actual code or configuration

#### Parameters

```json
{
  "path": {
    "type": "string",
    "description": "The file path to read (absolute or relative)",
    "required": true
  },
  "read_range": {
    "type": "array",
    "items": { "type": "number" },
    "minItems": 2,
    "maxItems": 2,
    "description": "Optional [start_line, end_line] to read only specific lines",
    "required": false
  },
  "repository": {
    "type": "string",
    "description": "Repository URL (e.g., https://github.com/owner/repo)",
    "required": true
  }
}
```

#### Features

- Returns file contents with line numbers
- Maximum file size: 128KB
- Supports read_range for reading specific line ranges
- Line-numbered output (e.g., "1: abc\n")

#### Implementation Logic

```javascript
// Lines 5044-5078 (rM8 function)
// 1. Parse repository URL to extract owner/repo
// 2. Clean up file path (remove file:// prefix, handle absolute paths)
// 3. Call GitHub API: repos/${owner}/${repo}/contents/${path}
// 4. Decode base64 content if necessary
// 5. Apply read_range filter if specified
// 6. Check file size limit (128KB)
// 7. Add line numbers to each line
// 8. Return formatted content
```

#### Examples

```
<example>
  <user>Read the main configuration file from the repository</user>
  <response>Calls the read tool with path: "package.json"</response>
</example>

<example>
  <user>Show me lines 50-100 of the authentication service</user>
  <response>Calls the read tool with path: "src/auth/service.ts", read_range: [50, 100]</response>
</example>

<example>
  <user>Read the README from https://github.com/owner/repo</user>
  <response>Calls the read tool with path: "README.md", repository: "https://github.com/owner/repo"</response>
</example>
```

---

### 2. Search Code (`search_github_code` / AO0)

**Variable:** `AO0` (Lines 5149)  
**Function:** `oM8` (Lines 5079-5149)

#### Description

Search for code patterns and content in repositories with structured results grouped by file.

#### When to Use
- When you need to find code patterns across a repository with contextual information
- When you want to understand how certain functionality is implemented across multiple files
- When you need to see code snippets with surrounding context
- When building comprehensive answers about code implementation

#### Parameters

```json
{
  "pattern": {
    "type": "string",
    "description": "The search pattern to find in code. Supports GitHub search operators (AND, OR, NOT) and qualifiers. Max 256 characters, max 5 operators.",
    "required": true
  },
  "path": {
    "type": "string",
    "description": "Optional path to limit search to specific directory or file pattern",
    "required": false
  },
  "repository": {
    "type": "string",
    "description": "Repository URL (e.g., https://github.com/owner/repo)",
    "required": true
  },
  "limit": {
    "type": "number",
    "description": "Maximum number of results to return (default: 30, max: 100)",
    "minimum": 1,
    "maximum": 100,
    "required": false
  },
  "offset": {
    "type": "number",
    "description": "Number of results to skip for pagination (must be divisible by limit)",
    "minimum": 0,
    "required": false
  }
}
```

#### Features

- Groups results by file for better organization
- Provides code chunks with surrounding context
- Returns structured data suitable for detailed analysis
- Supports GitHub search operators (AND, OR, NOT) with up to 5 operators per query
- Validates queries against GitHub API limits (256 character max)
- Supports pagination with limit and offset parameters
- Chunks truncated at ~2048 characters

#### Search Operators

- **AND**: Both terms must be present (e.g., "auth AND login")
- **OR**: Either term can be present (e.g., "function OR method")
- **NOT**: Exclude results with the term (e.g., "auth NOT test")
- Maximum 5 operators per query

#### GitHub Qualifiers

- `language:LANGUAGE` (e.g., "language:typescript")
- `path:PATH` (e.g., "path:src/components")
- `extension:EXT` (e.g., "extension:ts")
- `in:file` or `in:path` (search in file content vs filename)

#### Result Structure

```json
{
  "results": [
    {
      "file": "file path",
      "chunks": ["code chunk 1", "code chunk 2"]
    }
  ],
  "totalCount": 42
}
```

#### Implementation Logic

```javascript
// Lines 5079-5149 (oM8 function)
// 1. Validate offset is divisible by limit
// 2. Parse repository URL
// 3. Construct GitHub search query: pattern + repo + path
// 4. Call GitHub API: search/code with text-match highlighting
// 5. Group results by file path
// 6. Extract text_matches fragments (content chunks)
// 7. Truncate chunks over 2048 characters
// 8. Return structured results with totalCount
```

#### Examples

```
<example>
  <user>Find the handleAuth function definition in the src directory</user>
  <response>Calls the search tool with pattern: "function handleAuth path:src"</response>
</example>

<example>
  <user>Find the UserManager class definition in TypeScript files</user>
  <response>Calls the search tool with pattern: "class UserManager language:typescript"</response>
</example>

<example>
  <user>Find React imports but exclude test files</user>
  <response>Calls the search tool with pattern: "import from react NOT test"</response>
</example>
```

---

### 3. Search Commits (`search_github_commits` / FO0)

**Variable:** `FO0` (Referenced in tool array)  
**Description:** Search commits by various criteria (author, date, message, etc.)

*Note: Full implementation details would require additional code extraction from the minified file.*

---

### 4. Diff GitHub (`diff_github` / BO0)

**Variable:** `BO0` (Lines 4900-4929)  
**Function:** `hM8` (description) / implementation function

#### Description

Compare commits, branches, or tags and show file-level changes with optional diff patches.

#### When to Use
- When you need to see what changed between two commits, branches, or tags
- When investigating code changes for a feature or bug fix
- When reviewing the impact of a merge or pull request
- When understanding code evolution

#### Parameters

```json
{
  "base": {
    "type": "string",
    "description": "The base commit SHA, branch name, or tag to compare from (e.g., 'main', 'v1.0.0', or commit SHA)",
    "required": true
  },
  "head": {
    "type": "string",
    "description": "The head commit SHA, branch name, or tag to compare to (e.g., 'feature-branch', 'v2.0.0', or commit SHA)",
    "required": true
  },
  "repository": {
    "type": "string",
    "description": "Repository URL (e.g., https://github.com/owner/repo)",
    "required": true
  },
  "includePatches": {
    "type": "boolean",
    "description": "Include unified diff patches per file (token heavy, truncated to ~4k characters per file). Default false.",
    "required": false
  }
}
```

#### Features

- Returns detailed file-level change information with statistics
- Includes optional line-by-line diff patches (token-heavy, controlled by includePatches parameter)
- Supports comparing commits, branches, and tags
- Shows file status (added, removed, modified, renamed, etc.)
- Patches automatically truncated at ~4k characters to save tokens
- Note: GitHub may omit patches for binary or very large files

#### Examples

```
<example>
  <user>Show me what changed between main and the feature-auth branch</user>
  <response>Calls the diff tool with base: "main", head: "feature-auth"</response>
</example>

<example>
  <user>What files were modified in commit abc123 compared to its parent?</user>
  <response>Calls the diff tool with base: "abc123^", head: "abc123"</response>
</example>

<example>
  <user>Compare version v1.0.0 to v2.0.0</user>
  <response>Calls the diff tool with base: "v1.0.0", head: "v2.0.0"</response>
</example>
```

---

### 5. Glob GitHub (`glob_github` / MO0)

**Variable:** `MO0` (Lines 4929-4967)  
**Function:** `gM8` (Lines 4929+)

#### Description

Find files matching a glob pattern in a repository.

#### When to Use
- When you need to find specific file types (e.g., all JavaScript files)
- When you want to find files in specific directories or following specific patterns
- When you need to explore the codebase structure quickly

#### Parameters

```json
{
  "filePattern": {
    "type": "string",
    "description": "Glob pattern to match files (e.g., '**/*.ts', 'src/**/*.test.js')",
    "required": true
  },
  "limit": {
    "type": "number",
    "description": "Maximum number of results to return (default: 100)",
    "required": false
  },
  "offset": {
    "type": "number",
    "description": "Number of results to skip for pagination",
    "required": false
  },
  "repository": {
    "type": "string",
    "description": "Repository URL (e.g., https://github.com/owner/repo)",
    "required": true
  }
}
```

#### Pattern Examples

- `**/*.js` - All JavaScript files in any directory
- `src/**/*.ts` - All TypeScript files under the src directory
- `*.json` - All JSON files in the current directory
- `**/*test*` - All files with "test" in their name
- `web/src/**/*` - All files under the web/src directory
- `**/*.{js,ts}` - All JavaScript and TypeScript files
- `src/[a-z]*/*.ts` - TypeScript files in src subdirectories that start with lowercase letters

#### Implementation Logic

```javascript
// Lines 4929+ (gM8 function)
// 1. Parse repository URL
// 2. Call GitHub API: repos/${owner}/${repo}/git/trees/HEAD?recursive=1
// 3. Filter tree for blob types (files)
// 4. Apply minimatch glob pattern matching
// 5. Paginate results based on limit and offset
// 6. Return list of matching file paths
```

#### Examples

```
<example>
  <user>Find all TypeScript test files</user>
  <response>Calls the glob tool with filePattern: "**/*.test.ts"</response>
</example>

<example>
  <user>List all configuration files in the root</user>
  <response>Calls the glob tool with filePattern: "*.{json,yaml,yml,toml}"</response>
</example>

<example>
  <user>Find React components in https://github.com/owner/repo</user>
  <response>Calls the glob tool with filePattern: "**/*.tsx", repository: "https://github.com/owner/repo"</response>
</example>
```

---

### 6. List Directory (`list_directory` / VO0)

**Variable:** `VO0` (Lines 4967-4995)  
**Function:** `pM8` (Lines 4968+)

#### Description

List the contents of a directory in a GitHub repository.

#### When to Use
- When you need to understand the structure of a directory
- When exploring a codebase to find relevant files
- When you want to see what files and subdirectories exist in a specific location

#### Parameters

```json
{
  "path": {
    "type": "string",
    "description": "The directory path to list (absolute or relative, defaults to root)",
    "required": true
  },
  "repository": {
    "type": "string",
    "description": "Repository URL (e.g., https://github.com/owner/repo)",
    "required": true
  },
  "limit": {
    "type": "number",
    "description": "Maximum number of entries to return (default: 100, max: 1000)",
    "minimum": 1,
    "maximum": 1000,
    "required": false
  }
}
```

#### Features

- Returns list of files and directories
- Directories have trailing slash (/)
- Sorted alphabetically (directories first, then files)
- Default limit: 100 entries
- Maximum limit: 1000 entries

#### Implementation Logic

```javascript
// Lines 4968+ (pM8 function and uM8 helper)
// 1. Parse repository URL
// 2. Clean up path (remove file:// prefix, handle "." for root)
// 3. Call GitHub API: repos/${owner}/${repo}/contents/${path}
// 4. Extract file/directory names
// 5. Add trailing "/" for directories
// 6. Sort alphabetically (directories first)
// 7. Apply limit
// 8. Return list
```

#### Examples

```
<example>
  <user>List the contents of the src directory</user>
  <response>Calls the list_directory tool with path: "src"</response>
</example>

<example>
  <user>Show me what's in the root of the repository</user>
  <response>Calls the list_directory tool with path: "" or "."</response>
</example>

<example>
  <user>Explore the components folder in https://github.com/owner/repo</user>
  <response>Calls the list_directory tool with path: "src/components", repository: "https://github.com/owner/repo"</response>
</example>
```

---

### 7. List Repositories (`list_repositories` / NO0)

**Variable:** `NO0` (Lines 4995-5044)  
**Function:** `nM8` (Lines 4995+) with helpers `cM8`, `lM8`, `iM8`

#### Description

List and search for repositories using a hybrid approach that prioritizes user's own repositories.

#### When to Use
- When you need to find repositories based on name patterns
- When you want to explore repositories in a specific organization
- When you need to filter repositories by programming language
- When you need repository metadata (stars, forks, descriptions)

#### Parameters

```json
{
  "pattern": {
    "type": "string",
    "description": "Optional pattern to match in repository names",
    "required": false
  },
  "organization": {
    "type": "string",
    "description": "Optional organization name to filter repositories",
    "required": false
  },
  "language": {
    "type": "string",
    "description": "Optional programming language to filter repositories",
    "required": false
  },
  "limit": {
    "type": "number",
    "description": "Maximum number of repositories to return (default: 30, max: 100)",
    "minimum": 1,
    "maximum": 100,
    "required": false
  },
  "offset": {
    "type": "number",
    "description": "Number of results to skip for pagination (must be divisible by limit)",
    "minimum": 0,
    "required": false
  }
}
```

#### Features

- Hybrid search approach:
  1. First searches user's repositories (owned, collaborator, or organization member)
  2. If insufficient results, supplements with public GitHub repositories
  3. User's repositories always shown first for maximum relevance
- Search by repository name patterns
- Filter by organization
- Filter by programming language
- Sort by popularity (stars)
- Returns comprehensive repository metadata
- Ensures both relevance and discovery

#### Search Behavior

- User's repositories matching the criteria are shown first
- If insufficient results from user's repositories, public repositories are added to reach the limit
- Results sorted by star count within each category (user repos first, then public)

#### Result Structure

```json
{
  "repositories": [
    {
      "name": "owner/repo-name",
      "description": "Repository description",
      "language": "TypeScript",
      "stargazersCount": 1234,
      "forksCount": 56,
      "private": false
    }
  ],
  "totalCount": 42
}
```

#### Implementation Logic

```javascript
// Lines 4995+ (nM8, cM8, lM8, iM8 functions)
// 1. Validate offset divisible by limit
// 2. Fetch user's repositories (5x limit to ensure good coverage)
//    - API: user/repos?per_page=X&page=Y&sort=updated&affiliation=owner,collaborator,organization_member
// 3. Filter user repos by pattern, organization, and language (lM8)
// 4. Sort by star count
// 5. If insufficient results, search public repositories (iM8)
//    - API: search/repositories?q={pattern} org:{org} language:{lang}
// 6. Merge results (user repos first, deduplicated)
// 7. Return repositories with metadata and totalCount
```

#### Examples

```
<example>
  <user>Find repositories with "api" in the name in the "myorg" organization</user>
  <response>Calls the list_repositories tool with pattern: "api", organization: "myorg"</response>
</example>

<example>
  <user>List TypeScript repositories in the "facebook" organization</user>
  <response>Calls the list_repositories tool with organization: "facebook", language: "TypeScript"</response>
</example>

<example>
  <user>Find my repositories containing "frontend"</user>
  <response>Calls the list_repositories tool with pattern: "frontend"</response>
</example>
```

---

## Oracle Agent Tools

The Oracle agent has access to standard codebase analysis tools plus diagram rendering.

**Tool Array Variable:** `b_1` (Referenced in Oracle description, Line 5287)

### Oracle Tool List

1. **Read** - Read files from local workspace
2. **Grep** - Search for text patterns in files
3. **glob** - Find files by pattern matching
4. **web_search** - Search the web for information
5. **read_web_page** - Read and analyze web pages
6. **render_mermaid** (`QS`) - Render Mermaid diagrams

### Render Mermaid (`render_mermaid` / P96)

**Variable:** `P96` (Lines 5258-5279)  
**Description Variable:** `Go8` (Lines 5258-5279)

#### Description

Renders a Mermaid diagram from the provided code.

#### When to Use

PROACTIVELY USE DIAGRAMS when they would better convey information than prose alone.

Create diagrams WITHOUT being explicitly asked in these scenarios:
- When explaining system architecture or component relationships
- When describing workflows, data flows, or user journeys
- When explaining algorithms or complex processes
- When illustrating class hierarchies or entity relationships
- When showing state transitions or event sequences

#### Diagrams Are Especially Valuable For

- Application architecture and dependencies
- API interactions and data flow
- Component hierarchies and relationships
- State machines and transitions
- Sequence and timing of operations
- Decision trees and conditional logic

#### Parameters

```json
{
  "code": {
    "type": "string",
    "description": "The Mermaid diagram code to render (DO NOT override with custom colors or other styles)",
    "required": true
  }
}
```

#### Styling Guidelines

- When defining custom classDefs, always define fill color, stroke color, and text color ("fill", "stroke", "color") explicitly
- **IMPORTANT:** Use DARK fill colors (close to #000) with light stroke and text colors (close to #fff)

#### Implementation

```javascript
// Lines 5258-5279 (P96 variable and implementation)
// Uses Mermaid.js library to render diagrams
// Strips out <img>, <image>, <svg> tags from output
// Initializes with:
//   - startOnLoad: false
//   - securityLevel: "loose"
//   - suppressErrorRendering: true
// Returns success status or error message
```

---

## Smart Agent Tools

The Smart Agent (main orchestrator) has access to the full suite of tools available in the AMP CLI system.

### Core Tools

Based on the system prompts and tool descriptions, the Smart Agent has access to:

1. **File Operations**
   - `Read` - Read files with optional line ranges
   - `edit_file` - Make edits to files
   - `create_file` - Create new files
   - `format_file` - Format files using VS Code formatter
   - `undo_edit` - Undo last edit

2. **Search and Discovery**
   - `Grep` - Fast text pattern matching with ripgrep
   - `glob` - File pattern matching
   - `finder` / `xai_finder` - Intelligent codebase search
   - `codebase_search` - Semantic code search (mentioned as subagent option)

3. **Execution and Diagnostics**
   - `Bash` - Execute shell commands
   - `get_diagnostics` - Get errors and warnings for files/directories

4. **Web Tools**
   - `web_search` - Search the web
   - `read_web_page` - Read and analyze web pages

5. **Visualization**
   - `mermaid` - Render Mermaid diagrams

6. **Task Management**
   - `todo_read` - Read current todo list
   - `todo_write` - Update todo list

7. **Subagents**
   - `Task` - Fire-and-forget executor for multi-step implementations
   - `oracle` - Senior engineering advisor (GPT-5)
   - `librarian` - Multi-repository codebase expert
   - `finder` / `xai_finder` - Intelligent search agents

8. **MCP Integration**
   - `read_mcp_resource` - Read resources from MCP servers

---

## Tool Implementation Patterns

### Common Patterns

#### 1. Observable Pattern

Most tools return RxJS Observables that emit progress updates:

```javascript
return new t1((observer) => {
  // Emit in-progress status
  observer.next({status:"in-progress", progress:[...]});
  
  // Do work...
  
  // Emit done status with result
  observer.next({status:"done", result:...});
  observer.complete();
  
  // Or emit error
  observer.next({status:"error", error:{message:"..."}});
  observer.complete();
});
```

#### 2. GitHub API Helper

Tools use a centralized `uQ()` helper function for GitHub API calls:

```javascript
uQ(endpoint, options, {configService:Q})
```

This helper:
- Handles authentication
- Manages API endpoints
- Returns standardized responses with `{ok: boolean, data: any, status: number, statusText: string}`

#### 3. Repository URL Parsing

Tools consistently parse repository URLs:

```javascript
let repo = url
  .replace(/\.git$/, "")
  .replace("https://github.com/", "");
```

#### 4. Path Normalization

Tools normalize file paths to handle various formats:

```javascript
// Remove file:// prefix
if (path.startsWith("file://")) path = path.slice(7);

// Remove workspace root if present
if (path.startsWith(workspaceRoot)) path = path.slice(workspaceRoot.length);

// Remove leading slash
if (path.startsWith("/")) path = path.slice(1);
```

#### 5. Pagination Validation

Tools with pagination validate offset/limit alignment:

```javascript
if (offset % limit !== 0) {
  return error(`offset (${offset}) must be divisible by limit (${limit})`);
}
```

#### 6. Resource URI Construction

Tools build workspace URIs using a helper:

```javascript
let workspace = {
  uri: I6(w0.from({scheme:"file", path:`/${repo}`})),
  repository: {type:"git", url:repositoryUrl},
  fs:"github"
};
```

### Error Handling

Tools follow consistent error handling:

1. **API Failures**: Check `response.ok` and return descriptive error messages
2. **Validation Failures**: Return errors for invalid parameters
3. **Size Limits**: Check content size and return helpful error messages
4. **Authentication**: Handle authentication failures gracefully

### Tool Registration

Tools are registered with a tool service:

```javascript
let toolService = WO({configService: config});
for (let tool of toolArray) {
  registrations.push(toolService.registerTool(tool));
}
```

### Agent Orchestration

Agents subscribe to tool execution:

```javascript
let agent = new AgentClass(tools, systemPrompt, query, environment);
let subscription = agent.subscribe({
  next: (update) => handleUpdate(update),
  error: (err) => handleError(err),
  complete: () => handleComplete()
});
```

---

## Tool Schema Structure

All tools follow a consistent schema structure:

```javascript
{
  spec: {
    name: "tool_name",
    description: "Tool description with examples",
    inputSchema: {
      type: "object",
      properties: { /* parameters */ },
      required: [ /* required params */ ]
    },
    meta: {
      disableTimeout: true  // optional
    },
    source: "builtin"
  },
  fn: toolImplementationFunction
}
```

### Input Schema Format

```javascript
{
  type: "object",
  properties: {
    paramName: {
      type: "string|number|boolean|array",
      description: "Parameter description",
      minimum: 1,           // for numbers
      maximum: 100,         // for numbers
      items: {...},         // for arrays
      minItems: 2,          // for arrays
      maxItems: 2           // for arrays
    }
  },
  required: ["param1", "param2"]
}
```

---

## Summary

### Librarian Tools (7 GitHub Tools)

Specialized for multi-repository GitHub codebase analysis:
- read, search_github_code, search_github_commits, diff_github, glob_github, list_directory, list_repositories

### Oracle Tools (6 Standard Tools + 1 Visualization)

Standard codebase analysis plus visualization:
- Read, Grep, glob, web_search, read_web_page, render_mermaid

### Smart Agent Tools (15+ Tools)

Complete toolkit for software engineering tasks:
- File operations, search, execution, diagnostics, web, visualization, task management, subagents, MCP integration

### Key Differences

- **Librarian**: GitHub-focused, multi-repository exploration
- **Oracle**: Local workspace analysis + web research + visualization
- **Smart Agent**: Full environment control + orchestration capabilities
