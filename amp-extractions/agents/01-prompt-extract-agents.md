# AMP Agent System Prompt Extraction Request

I need you to extract and document the system prompts and tools for all agents from the AMP CLI minified JavaScript file located at `node_modules/@sourcegraph/amp/dist/main.js`.

You should always use the "newest" or "latest" installation of the amp cli from the `npm-packages/` directory. For example, `npm-packages/0.0.1761153678-gfa55cf/node_modules/@sourcegraph/amp/dist/main.js` if `0.0.1761153678-gfa55cf` is the newest version of the amp cli in `npm-packages/`

## Required Outputs

Create the following files in `amp-extractions/agents/`:

### 1. Smart Agent System Prompt
**File:** `/home/code/projects/ben-vargas/ai-amp-cli/main/amp-extractions/agents/smart-system-prompt.md`

Extract the complete system prompt for the main/smart/default agent. This is the primary agent that users interact with. In the minified code, this appears to be defined in the `z90` function (around line 399-623).

The prompt should include all sections:
- Role & Agency
- Guardrails
- Fast Context Understanding
- Parallel Execution Policy
- Tools and function calls
- TODO tool usage
- Subagents (Task, Oracle, Codebase Search)
- Quality Bar
- Verification Gates
- Markdown Formatting Rules
- Final Status Spec
- Working Examples
- Strict Concision rules

### 2. Librarian Agent System Prompt
**File:** `/home/code/projects/ben-vargas/ai-amp-cli/main/amp-extractions/agents/librarian-system-prompt.md`

Extract the complete system prompt for the Librarian agent. This is a specialized codebase understanding agent that works with GitHub repositories. In the minified code, this is defined in the `sM8` variable (around line 5188-5200+).

The prompt should include:
- Role description
- Key responsibilities
- Guidelines for tool usage
- Communication style and formatting
- Direct & detailed communication rules
- Linking guidelines for GitHub URLs

### 3. Oracle Agent System Prompt
**File:** `/home/code/projects/ben-vargas/ai-amp-cli/main/amp-extractions/agents/oracle-system-prompt.md`

Extract the complete system prompt for the Oracle agent. This is a reasoning-powered senior engineering advisor using GPT-5. In the minified code, this is defined in the `Xo8` variable (around line 5314-5320+).

The prompt should include:
- Role description
- Key responsibilities
- Operating principles (simplicity-first)
- Tool usage guidelines
- Response format structure (TL;DR, Recommended approach, Rationale, etc.)
- Guidelines for advice and recommendations

### 4. **NEW** Agents

If any new agents have been added that are not included above, create an appropriately named markdown file for the new agent and extract its full system prompt similar patterns in sections 1 through 3.

### 5. Complete Tool Documentation
**File:** `/home/code/projects/ben-vargas/ai-amp-cli/main/amp-extractions/agents/agent-tools.md`

Extract and document the complete tool definitions and implementations for each agent, including:

#### Librarian Agent Tools (7 GitHub tools):
1. **read** (`Nd`) - Read files from GitHub repositories
2. **search_github_code** (`Ld`) - Search code with grouped results
3. **search_github_commits** (`Ad`) - Search commits by various criteria
4. **diff_github** (`jd`) - Compare commits/branches/tags
5. **glob_github** (`Od`) - Find files by glob patterns
6. **list_directory** (`wd`) - List directory contents
7. **list_repositories** (`Rd`) - Hybrid repository search

For each tool, document:
- Tool name (both minified variable and readable name)
- Description
- When to use / When NOT to use
- Parameters (with types, required/optional, descriptions)
- Features
- Implementation code/logic (extracted from the minified file)
- Examples

#### Oracle Agent Tools:
- **render_mermaid** (`QS`) - Render diagrams for visualization
- Access to standard codebase tools (Read, Grep, glob, web_search, read_web_page)

#### Smart Agent Tools:
Document the tools available to the main/smart agent (this will likely be the full suite of tools including file operations, bash, diagnostics, etc.)

### 5. Tool Implementation Patterns
Include in `/home/code/projects/ben-vargas/ai-amp-cli/main/amp-extractions/agents/agent-tools.md`:
- Common implementation patterns (Observable pattern, GitHub API helper)
- Error handling approaches
- Tool registration and organization
- Agent orchestration details

## Extraction Guidelines

1. **Use the Oracle tool** to help parse the minified JavaScript file and extract the relevant sections
2. **Preserve exact text** - Do not summarize or paraphrase the system prompts; extract them verbatim
3. **Include implementation code** - For tools, include the actual JavaScript implementation logic (even if minified)
4. **Line number references** - Note the line numbers where each prompt/tool is defined in the source file
5. **Complete extraction** - Don't truncate long prompts; include everything
6. **Proper formatting** - Format as clean Markdown with appropriate headers and code blocks

## Process

1. First, use glob to locate the main.js file
2. Use the Oracle to identify the exact line ranges for:
   - Smart agent prompt (z90 function)
   - Librarian agent prompt (sM8 variable)
   - Oracle agent prompt (Xo8 variable)
   - Tool definitions (sw1 array for Librarian, T_1 array for Oracle)
3. Read those specific line ranges
4. Create the four markdown files with the extracted content
5. Ensure all prompts are complete and all tools are documented with their implementation logic

## Success Criteria

- ✅ All three agent system prompts are extracted verbatim
- ✅ All tools for each agent are documented with implementation code
- ✅ Line number references are provided for traceability
- ✅ Files are properly formatted as Markdown
- ✅ Documentation is comprehensive enough to understand how each agent works
- ✅ Tool schemas (parameters, types) are clearly documented
- ✅ Usage guidelines and examples are included for each tool

## Additional Context

The AMP CLI uses these three agents in a hierarchical system:
- **Smart Agent**: The main orchestrator that users interact with directly
- **Librarian**: A subagent for deep multi-repository GitHub code analysis
- **Oracle**: A subagent for reasoning-intensive tasks like architecture review and planning

Each agent has its own system prompt, tool set, and behavioral characteristics. The goal is to fully document all three so they can be understood, replicated, or modified in the future.
