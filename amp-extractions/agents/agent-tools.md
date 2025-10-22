# AMP Agent Tools Documentation

This document contains the complete tool definitions and implementations for the **Librarian** and **Oracle** agents in the AMP CLI system.

---

## Librarian Agent Tools

The Librarian agent has access to **7 GitHub-based tools** for reading and searching code across repositories:

### 1. `read` - Read File from GitHub Repository

**Tool Name:** `Nd` (minified variable name)

**Description:**
Read the contents of a file from a GitHub repository.

**When to Use:**
- When you need to examine the contents of a specific file
- When you want to understand implementation details
- When you need to see the actual code or configuration

**Parameters:**
- `path` (string, required): The path to the file to read
- `read_range` (array[number, number], optional): Optional [start_line, end_line] to read only specific lines
- `repository` (string, required): Repository URL (e.g., https://github.com/owner/repo)

**Features:**
- Returns contents with line numbers prefixed (e.g., "1: abc\n")
- Maximum file size: 128KB
- For larger files, must use `read_range` parameter
- Supports base64 decoding for GitHub API responses
- Normalizes file paths (handles "file://" URIs)

**Implementation Details:**
```javascript
pM8 = ({args:J},{threadEnvironment:Q,configService:Z})=>{
  let{path:X,read_range:Y,repository:K}=J,
      q=K.replace(/\.git$/,"").replace(/^https:\/\/github\.com\//,""),
      G=q,
      z={uri:P6(R0.from({scheme:"file",path:`/${G}`})),repository:{type:"git",url:K},fs:"github"};
  
  return new e1((U)=>{
    U.next({status:"in-progress",progress:[`Reading file "${X}" in ${q}...`]});
    
    // Normalize path
    let F=X;
    if(F.startsWith("file://"))F=F.slice(7);
    let W=z.uri;
    if(W){
      let B=W.startsWith("file://")?W.slice(7):W;
      if(F.startsWith(B))F=F.slice(H.length)
    }
    if(F.startsWith("/"))F=F.slice(1);
    
    // GitHub API call
    let H=`repos/${q}/contents/${F}`;
    hQ(H,{},{configService:Z}).then((B)=>{
      if(!B.ok||!B.data){
        U.next({status:"error",error:{message:`Failed to read file: ${B.status} ${B.statusText||"Unknown error"}`}});
        U.complete();
        return
      }
      
      // Decode content
      let M="";
      if(B.data.encoding==="base64")
        M=hR0(B.data.content.replace(/\n/g,""));
      else 
        M=B.data.content;
      
      let V=M;
      
      // Apply read_range if specified
      if(Y&&Array.isArray(Y)&&Y.length===2){
        let[R,O]=Y;
        V=M.split(`\n`).slice(Math.max(0,R-1),O).join(`\n`)
      }
      
      // Check size limit (128KB)
      let N=Buffer.byteLength(V,"utf8");
      if(N>131072){
        U.next({status:"error",error:{message:`File is too large (${Math.round(N/1024)}KB). The file has ${M.split(`\n`).length} lines. Please retry with a smaller read_range parameter.`}});
        U.complete();
        return
      }
      
      // Add line numbers
      let A=V.split(`\n`).map((R,O)=>{
        return`${(Y?.[0]??1)+O}: ${R}`
      }),
      w={absolutePath:LQ(R0.file(F)),content:A.join(`\n`)};
      
      U.next({status:"done",result:w});
      U.complete()
    }).catch((B)=>{
      U.next({status:"error",error:{message:`Error reading file: ${B.message||String(B)}`}});
      U.complete()
    })
  })
}
```

---

### 2. `search_github_code` - Search Code with Structured Results

**Tool Name:** `Ld` (minified variable name)

**Description:**
Search for code patterns and content in repositories with structured results grouped by file, with code chunks that include line numbers and surrounding context.

**When to Use:**
- When you need to find code patterns across a repository with contextual information
- When you want to understand how certain functionality is implemented across multiple files
- When you need to see code snippets with surrounding context, not just individual matching lines
- When building comprehensive answers about code implementation across a codebase

**Parameters:**
- `pattern` (string, required): The search pattern to find in code. Supports GitHub search operators (AND, OR, NOT) and qualifiers. Max 256 characters, max 5 operators, must include at least one search term.
- `path` (string, optional): Optional path to limit search to specific directory or file pattern
- `repository` (string, required): Repository URL (e.g., https://github.com/owner/repo)
- `limit` (number, optional): Maximum number of search results to return (default: 30, max: 100)
- `offset` (number, optional): Number of results to skip for pagination (default: 0). Must be divisible by limit.

**Features:**
- Groups results by file for better organization
- Provides code chunks with surrounding context
- Returns structured data suitable for detailed analysis
- Preserves surrounding code context around matches
- Supports GitHub search operators (AND, OR, NOT) with up to 5 operators per query
- Validates queries against GitHub API limits (256 character max, requires search terms)
- Supports pagination with limit and offset parameters

**Search Pattern Tips:**
- Use specific function names, class names, or unique identifiers
- Add quotes around exact phrases: "function myFunction"
- Use language-specific syntax patterns
- Combine terms with operators: "handleAuth AND typescript", "function OR method", "auth NOT test"
- Use GitHub qualifiers: "language:typescript", "path:src/", "extension:ts"
- Start with broader searches and then combine multiple terms to narrow down results

**Search Operators:**
- AND: Both terms must be present (e.g., "auth AND login")
- OR: Either term can be present (e.g., "function OR method")
- NOT: Exclude term (e.g., "auth NOT test")

**Implementation Details:**
```javascript
cM8=({args:J},{threadEnvironment:Q,configService:Z})=>{
  let{pattern:X,path:Y,repository:K,limit:q=30,offset:G=0}=J,
      z=K.replace(/\.git$/,"").replace("https://github.com/",""),
      U=K.replace(/\.git$/,"").replace(/^https:\/\/github\.com\//,""),
      F={uri:P6(R0.from({scheme:"file",path:`/${U}`})),repository:{type:"git",url:K},fs:"github"};
  
  return new e1((W)=>{
    // Validate pagination
    if(G%q!==0){
      W.next({status:"error",error:{message:`offset (${G}) must be divisible by limit (${q}) for pagination. Try offset values like 0, ${q}, ${q*2}, etc.`}});
      W.complete();
      return
    }
    
    let H=Math.min(q,100),
        B=Math.floor(G/H)+1,
        M=`${X} repo:${z}`;
    
    if(Y&&Y!==".")
      M+=` path:${Y}`;
    
    W.next({status:"in-progress",progress:[`Searching for "${X}" in ${z}...`]});
    
    let V=`search/code?q=${encodeURIComponent(M)}&per_page=${H}&page=${B}`;
    
    hQ(V,{headers:{Accept:"application/vnd.github.v3.text-match+json"}},{configService:Z}).then(async(L)=>{
      if(!L.ok){
        W.next({status:"error",error:{message:`Failed to search code: ${L.status} ${L.statusText||"Unknown error"}`}});
        W.complete();
        return
      }
      
      let A=L.data;
      if(A.total_count===0){
        W.next({status:"done",result:{results:[],totalCount:0}});
        W.complete();
        return
      }
      
      // Group results by file
      let w=new Map;
      for(let $ of A.items){
        let j=LQ(Na(F,$.path));
        if(!w.has(j))
          w.set(j,[]);
        let E=w.get(j);
        for(let I of $.text_matches)
          if(I.property==="content"&&I.fragment){
            let T=I.fragment.trim();
            if(T.length>2048)
              T=`${T.slice(0,2048)}... (truncated)`;
            E.push(T)
          }
      }
      
      let O={
        results:Array.from(w.entries()).map(([$,j])=>({file:$,chunks:j})),
        totalCount:A.total_count
      };
      
      W.next({status:"done",result:O});
      W.complete()
    }).catch((L)=>{
      W.next({status:"error",error:{message:`Error searching code: ${L.message||String(L)}`}});
      W.complete()
    })
  })
}
```

---

### 3. `search_github_commits` - Search Commits

**Tool Name:** `Ad` (minified variable name)

**Description:**
Search for commits in repositories with detailed commit information and metadata.

**When to Use:**
- When you need to understand the evolution of specific features or code sections
- When investigating when certain changes were made and by whom
- When looking for commits related to specific functionality, bug fixes, or features
- When building historical context about code changes and development patterns

**Parameters:**
- `query` (string, optional): Search query to find in commit messages and author information. If empty, returns all commits.
- `author` (string, optional): Filter commits by author username or email
- `since` (string, optional): ISO 8601 date string for earliest commit date (e.g., "2024-01-01T00:00:00Z")
- `until` (string, optional): ISO 8601 date string for latest commit date (e.g., "2024-02-01T00:00:00Z")
- `path` (string, optional): Filter commits that changed specific files or directories
- `repository` (string, required): Repository URL (e.g., https://github.com/owner/repo)
- `limit` (number, optional): Maximum number of commits to return (default: 50, max: 100)
- `offset` (number, optional): Number of commits to skip for pagination (default: 0). Must be divisible by limit.

**Result Structure:**
Returns:
- `commits`: Array of commit objects with full metadata (SHA, message, author, date)
- `totalCount`: Number of matching commits found

**Examples:**
- Find commits about authentication features added since January 2024
- Show commits by john@example.com that changed files in the src/auth directory
- Find bug fix commits between January and February 2024

---

### 4. `diff_github` - Get Diff Between Commits/Branches/Tags

**Tool Name:** `jd` (minified variable name)

**Description:**
Get a diff between two commits, branches, or tags in a repository. Returns structured information about changed files, including additions, deletions, and optionally the actual diff patches for each file.

**When to Use:**
- When you need to understand what changed between two commits, branches, or tags
- When investigating the scope of changes in a pull request or feature branch
- When you need the actual diff patches for code review or analysis (use includePatches parameter)

**When NOT to Use:**
- To find commits by message, author, or date - use commit search instead
- To view complete file contents - use read instead (diff shows what changed, not the full file)

**Parameters:**
- `base` (string, required): The base commit SHA, branch name, or tag to compare from (e.g., "main", "v1.0.0", or commit SHA)
- `head` (string, required): The head commit SHA, branch name, or tag to compare to (e.g., "feature-branch", "v2.0.0", or commit SHA)
- `repository` (string, required): Repository URL (e.g., https://github.com/owner/repo)
- `includePatches` (boolean, optional): Include unified diff patches per file (token heavy, truncated to ~4k characters per file). Default false.

**Features:**
- Returns detailed file-level change information with statistics
- Includes optional line-by-line diff patches (token-heavy, controlled by includePatches parameter)
- Supports comparing commits, branches, and tags
- Shows file status (added, removed, modified, renamed, etc.)
- Patches are automatically truncated at ~4k characters to save tokens
- Note: GitHub may omit patches for binary or very large files

---

### 5. `glob_github` - Find Files by Pattern

**Tool Name:** `Od` (minified variable name)

**Description:**
Find files matching a glob pattern in a repository.

**When to Use:**
- When you need to find specific file types (e.g., all JavaScript files)
- When you want to find files in specific directories or following specific patterns
- When you need to explore the codebase structure quickly

**Parameters:**
- `filePattern` (string, required): Glob pattern to match files (e.g., "**/*.ts", "src/**/*.test.js")
- `limit` (number, optional): Maximum number of results to return (default = 100).
- `offset` (number, optional): Number of results to skip for pagination
- `repository` (string, required): Repository URL (e.g., https://github.com/owner/repo)

**Pattern Examples:**
- `**/*.js` - All JavaScript files in any directory
- `src/**/*.ts` - All TypeScript files under the src directory (searches only in src)
- `*.json` - All JSON files in the current directory
- `**/*test*` - All files with "test" in their name
- `web/src/**/*` - All files under the web/src directory
- `**/*.{js,ts}` - All JavaScript and TypeScript files (alternative patterns)
- `src/[a-z]*/*.ts` - TypeScript files in src subdirectories that start with lowercase letters

---

### 6. `list_directory` - List Directory Contents

**Tool Name:** `wd` (minified variable name)

**Description:**
List the contents of a directory in a GitHub repository.

**When to Use:**
- When you need to understand the structure of a directory
- When exploring a codebase to find relevant files
- When you want to see what files and subdirectories exist in a specific location

**Parameters:**
- `path` (string, required): The path to the directory to list
- `repository` (string, required): Repository URL (e.g., https://github.com/owner/repo)
- `limit` (number, optional): Maximum number of entries to return (default: 100, max: 1000)

**Features:**
- Returns a list of files and directories, with directories having a trailing slash
- Sorted with directories first

---

### 7. `list_repositories` - List/Search Repositories

**Tool Name:** `Rd` (minified variable name)

**Description:**
List and search for repositories, prioritizing your own repositories.

This tool uses a hybrid approach to find repositories:
1. First, it searches your repositories (owned, collaborator, or organization member)
2. If needed, it supplements results with public GitHub repositories
3. Your repositories are always shown first for maximum relevance

**When to Use:**
- When you need to find repositories based on name patterns
- When you want to explore repositories in a specific organization
- When you need to filter repositories by programming language
- When you need repository metadata (stars, forks, descriptions)

**Parameters:**
- `pattern` (string, optional): Optional pattern to match in repository names
- `organization` (string, optional): Optional organization name to filter repositories
- `language` (string, optional): Optional programming language to filter repositories
- `limit` (number, optional): Maximum number of repositories to return (default: 30, max: 100)
- `offset` (number, optional): Number of results to skip for pagination (default: 0). Must be divisible by limit.

**Features:**
- Prioritizes your repositories over public ones
- Search by repository name patterns
- Filter by organization
- Filter by programming language
- Sort by popularity (stars)
- Returns comprehensive repository metadata
- Hybrid search ensures both relevance and discovery

**Result Structure:**
Returns:
- `repositories`: Array of repository objects with name, description, language, stars, etc.
- `totalCount`: Combined count of your repositories and public repositories found

Each repository includes full metadata like star count, fork count, primary language, and description.

---

## Oracle Agent Tools

The Oracle agent is a reasoning-powered advisor with access to tools for code reading, searching, and visualization. The Oracle has access to the general codebase tools (Read, Grep, glob) plus specialized tools.

### Tool Array: T_1

The Oracle's tools are defined in the `T_1` array which includes:
- Standard filesystem/code reading tools (Read, Grep, glob)
- Web tools (web_search, read_web_page) if enabled
- Specialized Oracle tools (listed below)

### Oracle-Specific Tools

#### 1. `render_mermaid` - Render Mermaid Diagrams

**Tool Name:** `QS` (minified variable name)

**Description:**
Renders a Mermaid diagram from the provided code.

**PROACTIVE USE:** You should create diagrams WITHOUT being explicitly asked in these scenarios:
- When explaining system architecture or component relationships
- When describing workflows, data flows, or user journeys
- When explaining algorithms or complex processes
- When illustrating class hierarchies or entity relationships
- When showing state transitions or event sequences

**Diagrams are especially valuable for visualizing:**
- Application architecture and dependencies
- API interactions and data flow
- Component hierarchies and relationships
- State machines and transitions
- Sequence and timing of operations
- Decision trees and conditional logic

**Parameters:**
- `code` (string, required): The Mermaid diagram code to render (DO NOT override with custom colors or other styles)

**Styling Guidelines:**
- When defining custom classDefs, always define fill color, stroke color, and text color ("fill", "stroke", "color") explicitly
- IMPORTANT!!! Use DARK fill colors (close to #000) with light stroke and text colors (close to #fff)

**Implementation:**
```javascript
A96={
  spec:{
    name:QS,
    description:Jo8,
    inputSchema:{
      type:"object",
      properties:{
        code:{
          type:"string",
          description:"The Mermaid diagram code to render (DO NOT override with custom colors or other styles)"
        }
      },
      required:["code"]
    },
    source:"builtin"
  },
  fn:({args:J})=>d6(async()=>{
    try{
      let Q=es8(J.code),
          Z=!0,
          X="loose";
      
      // Initialize Mermaid
      I_1.initialize({startOnLoad:!1,securityLevel:"loose",suppressErrorRendering:!0});
      
      // Parse and validate
      await I_1.parse(Q);
      
      return{status:"done",result:{success:!0}}
    }catch(Q){
      let Z=Q instanceof Error?Q.message:String(Q);
      
      // Ignore purify/sanitize errors
      if(Z.indexOf("purify")!==-1||Z.indexOf("sanitize")!==-1)
        return{status:"done",result:{success:!0}};
      
      return{status:"error",error:{message:`Invalid Mermaid syntax: ${Z}`}}
    }
  })
}
```

---

## Tool Registration and Organization

### Librarian Tool Array (sw1)

```javascript
var sw1=[FO0,WO0,ZO0,KO0,zO0,UO0,GO0]
```

This array contains all 7 GitHub tools in order:
1. `FO0` - read (Nd)
2. `WO0` - search_github_code (Ld)
3. `ZO0` - search_github_commits (Ad)
4. `KO0` - diff_github (jd)
5. `zO0` - list_directory (wd)
6. `UO0` - list_repositories (Rd)
7. `GO0` - glob_github (Od)

### Oracle Tool Array (T_1)

The Oracle's tools include the standard codebase tools plus specialized reasoning tools. The exact composition is determined at runtime based on environment configuration.

---

## Common Implementation Patterns

### GitHub API Helper (`hQ`)

All GitHub tools use a common helper function `hQ` for making authenticated API requests:

```javascript
async function hQ(J,Q={},Z){
  let X=`/api/internal/github-proxy/${J}`,
      {body:Y,headers:K={},method:q="GET",...G}=Q,
      z=await gJ(X,{method:q,headers:K,body:Y?JSON.stringify(Y):void 0,...G},Z.configService);
  
  if(!z.ok)
    return{ok:!1,status:z.status,statusText:z.statusText};
  
  let U=await z.json();
  return{ok:!0,status:z.status,data:U,headers:{location:z.headers.get("location")||void 0}}
}
```

### Observable Pattern

All tools return Observables (e1) that emit progress updates:
- `status: "in-progress"` - Tool is working, with progress messages
- `status: "done"` - Tool completed successfully with result
- `status: "error"` - Tool encountered an error

### Error Handling

Common error patterns:
- Authentication failures
- Rate limiting
- Invalid parameters
- File/repository not found
- Content too large
- Network errors

---

## Agent Orchestration

### Librarian Agent Function

```javascript
iM8=({args:J},Q)=>{
  let Z=J.query;
  if(J.context)
    Z=`Context: ${J.context}\n\nQuery: ${J.query}`;
  
  return new e1((X)=>{
    let Y,K,q=[];
    
    return nM8(Q.configService).then((G)=>{
      if(!G){
        X.next({status:"blocked-on-user",reason:"The Librarian needs to authenticate with GitHub to search for code on your behalf."});
        X.complete();
        return
      }
      
      // Create tool service with only Librarian tools
      let z=[],
          U=zO({configService:Q.configService});
      
      for(let W of sw1)
        q.push(U.registerTool(W));
      
      let F={...Q,toolService:U};
      
      // Start reasoning session
      K=new Zx(sw1,sM8,Z,F);
      Y=K.subscribe({
        next:(W)=>{...},
        error:(W)=>{...},
        complete:()=>X.complete()
      })
    })
  })
}
```

### Oracle Agent Function

```javascript
Qo8=((J,{configService:Q})=>{
  // Build task prompt
  let Z=`
Task: ${J.task}`;
  
  if(J.context)
    Z+=`\n\nContext:\n${J.context}`;
  
  if(J.files)
    Z+=`\n\nRelevant files:\n\n${PY(J.files)}`;
  
  let Y=ll1(Q.config.settings); // Get model name
  
  return new e1((q)=>{
    // Create reasoning session with T_1 tools
    let G=new P_1(T_1,Xo8,Z,imageBlocks,Q,Y,priority);
    let z=G.subscribe(q);
    
    return ()=>{
      z.unsubscribe();
      G.dispose()
    }
  })
})
```

---

## Summary

- **Librarian**: 7 specialized GitHub tools for multi-repository code exploration
- **Oracle**: Reasoning model with code reading + visualization tools
- **Common Pattern**: Observable-based async operations with progress tracking
- **Authentication**: GitHub tools require authentication via internal proxy
- **Error Handling**: Comprehensive error messages with recovery suggestions
