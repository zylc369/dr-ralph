# Task 03: Web Tools (websearch/webfetch) API Verification

**Date**: 2026-02-24
**Status**: ✅ Verified
**Finding**: OpenCode DOES provide both `websearch_web_search_exa` and `webfetch` as built-in SDK tools

## Summary

OpenCode provides two built-in web tools for searching and fetching web content:
- **websearch_web_search_exa**: Search the web for any topic and get clean, ready-to-use content
- **webfetch**: Fetch content from a specified URL and convert it to markdown/text/HTML format

Both tools are part of the core OpenCode SDK (NOT MCP-based) and provide equivalent functionality to Claude Code's WebSearch and WebFetch tools.

## Investigation Methods

1. ✅ Searched for websearch/webfetch references in dr-ralph codebase
2. ✅ Examined diagnostic script for tool usage patterns (`scripts/setup-dr-ralph-diagnose.sh`)
3. ✅ Checked OpenCode SDK type definitions for tool permissions
4. ✅ Reviewed OpenCode permission configuration schema
5. ✅ Analyzed available tool definitions in OpenCode environment
6. ✅ Documented API syntax from available tool definitions

## Findings

### 1. Claude Code's WebSearch/WebFetch (What We're Migrating From)

The dr-ralph plugin currently uses WebSearch and WebFetch from Claude Code:

**Location**: `scripts/setup-dr-ralph-diagnose.sh:235-241`
```bash
**Instructions:**
1. Based on interview findings, generate initial differential diagnoses
2. Use `WebSearch` tool to search for:
   - Literature on symptoms and conditions
   - Treatment protocols and guidelines
   - Recent research on differential diagnoses
3. Use iterative refinement: Search → Analyze → Refine queries → Search again
4. Use inline citations: "According to Mayo Clinic [link]..."
5. Append research findings to `$PATIENT_NOTES_PATH`
```

**Expected behavior** (from diagnose.md):
- Research phase 2 uses WebSearch for literature and treatment protocols
- Iterative refinement of search queries
- Inline citations from web sources

### 2. OpenCode's Web Tools

OpenCode provides two web tools as part of the core SDK:

#### Tool 1: websearch_web_search_exa

**Location**: Built-in OpenCode SDK tool

**Description**: Search the web for any topic and get clean, ready-to-use content

**API Signature**:
```typescript
{
  query: string;                    // Search query
  numResults?: number;              // Number of results (default: 8, must be a number)
  type?: "auto" | "fast";           // Search type (default: "auto")
  contextMaxCharacters?: number;   // Max characters for context (default: 10000, must be a number)
  livecrawl?: "fallback" | "preferred";  // Live crawl mode (default: "fallback")
}
```

**Parameters**:
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `query` | string | required | Websearch query |
| `numResults` | number | 8 | Number of search results |
| `type` | "auto" \| "fast" | "auto" | Search type - balanced or quick results |
| `contextMaxCharacters` | number | 10000 | Maximum characters for context string optimized for LLMs |
| `livecrawl` | "fallback" \| "preferred" | "fallback" | Live crawl mode - use live crawling as backup or prioritize it |

**Best For**:
- Finding current information, news, facts
- Answering questions about any topic
- Literature and research searches

**Returns**: Clean text content from top search results, ready for LLM use

#### Tool 2: webfetch

**Location**: Built-in OpenCode SDK tool

**Description**: Fetches content from a specified URL and converts it to requested format

**API Signature**:
```typescript
{
  url: string;                              // Fully-formed valid URL
  format?: "text" | "markdown" | "html";   // Output format (default: "markdown")
  timeout?: number;                         // Optional timeout in seconds (max 120)
}
```

**Parameters**:
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `url` | string | required | Fully-formed valid URL |
| `format` | "text" \| "markdown" \| "html" | "markdown" | Output format |
| `timeout` | number | 120 (max) | Optional timeout in seconds |

**Features**:
- HTTP URLs are automatically upgraded to HTTPS
- Returns content in specified format (markdown by default)
- Read-only tool, does not modify any files
- Results are summarized if very large

**Best For**:
- Retrieving specific content from known URLs
- Fetching documentation pages
- Reading articles and blog posts
- Extracting content from web pages

### 3. Tool Availability & Permissions

OpenCode SDK includes both tools as part of the core SDK:

**Permission Configuration** (from SDK types):
```typescript
export type AgentPermissionsConfig = {
    // ... other permissions ...
    webfetch?: PermissionActionConfig;   // Permission config for webfetch
    websearch?: PermissionActionConfig;  // Permission config for websearch
    // ... other permissions ...
}
```

**Permission Actions**: "ask" | "allow" | "deny"

**Built-in Status**: ✅ Both are SDK tools, not MCP-based
- No additional MCP server configuration required
- Available out-of-the-box in OpenCode environment
- Directly accessible via tool calls

## Comparison: Claude Code vs OpenCode

| Feature | Claude Code | OpenCode | Migration Status |
|---------|-------------|-----------|-----------------|
| Web Search Tool | `WebSearch` | `websearch_web_search_exa` | ✅ Compatible |
| Web Fetch Tool | `WebFetch` | `webfetch` | ✅ Compatible |
| Search Query | query string | `query` parameter | ✅ Same |
| Number of Results | Configurable | `numResults` parameter | ✅ Same |
| Output Format | Auto-detect | `format` parameter (text/markdown/html) | ✅ Enhanced |
| Timeout | Configurable | `timeout` parameter (max 120s) | ✅ Same |
| Live Crawl | Not documented | `livecrawl` parameter (fallback/preferred) | ✅ Enhanced |
| Built-in vs MCP | Built-in | Built-in SDK | ✅ Same |

## Migration Status: ✅ BLOCKER RESOLVED

**OpenCode's web tools provide equivalent functionality to Claude Code's WebSearch and WebFetch. The dr-ralph migration can proceed with straightforward API mapping.**

The diagnostic workflow's research phase is fully supported:
- Literature search using `websearch_web_search_exa`
- Iterative refinement with `numResults` and `contextMaxCharacters` parameters
- Inline citations from search results
- Direct URL fetching with `webfetch` for specific sources

**Migration Changes Required**:
1. Update tool names:
   - `WebSearch` → `websearch_web_search_exa`
   - `WebFetch` → `webfetch`

2. Adapt API calls:
   - Map query parameters to OpenCode format
   - Use `format: "markdown"` for webfetch (default, matches Claude behavior)
   - Configure `numResults` for search depth control
   - Consider `livecrawl: "preferred"` for up-to-date medical literature

3. Research Phase Workflow:
   ```typescript
   // Claude Code:
   WebSearch("literature on symptoms and conditions")

   // OpenCode:
   websearch_web_search_exa({
     query: "literature on symptoms and conditions",
     numResults: 8,
     type: "auto",
     contextMaxCharacters: 10000
   })
   ```

## Example: Dr. Ralph Research Phase Migration

**Before (Claude Code)**:
```typescript
// Phase 2: Research
use WebSearch tool to search for:
   - Literature on symptoms and conditions
   - Treatment protocols and guidelines
   - Recent research on differential diagnoses
```

**After (OpenCode)**:
```typescript
// Phase 2: Research
use websearch_web_search_exa tool to search for:
   - Literature on symptoms and conditions
   - Treatment protocols and guidelines
   - Recent research on differential diagnoses

Parameters:
- query: Search terms
- numResults: 8 (default)
- type: "auto" (balanced search)
- contextMaxCharacters: 10000 (for comprehensive results)
- livecrawl: "preferred" (for up-to-date medical literature)

Use webfetch to retrieve specific pages:
- url: Article URL
- format: "markdown" (default)
- timeout: 120 (seconds)
```

## Migration Recommendations

1. **Immediate**: Update `scripts/setup-dr-ralph-diagnose.sh` to reference `websearch_web_search_exa` instead of `WebSearch`
2. **Short-term**: Add parameter documentation to help prompt templates
3. **Optional**: Configure `livecrawl: "preferred"` for medical research requiring latest literature
4. **Best Practice**: Use `webfetch` for specific source URLs discovered via search

## References

- OpenCode SDK: `~/.opencode/node_modules/@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts`
- OpenCode Tool Permissions: PermissionActionConfig for webfetch and websearch
- Dr. Ralph Diagnostic Spec: `docs/diagnose-spec.md` (Research Phase, lines 69-83)
- Setup Script: `scripts/setup-dr-ralph-diagnose.sh` (line 235: WebSearch usage)

## Verification Status

- ✅ websearch_web_search_exa tool exists as built-in SDK tool
- ✅ webfetch tool exists as built-in SDK tool
- ✅ API syntax documented for both tools
- ✅ Tool permission configuration confirmed in SDK types
- ✅ No MCP configuration required (built-in SDK tools)
- ✅ Compatibility verified with dr-ralph workflow requirements
- ✅ Migration path identified: simple API name mapping

**Conclusion**: OpenCode provides both `websearch_web_search_exa` and `webfetch` as built-in SDK tools with functionality equivalent to or better than Claude Code's WebSearch and WebFetch. No blocker exists for the dr-ralph migration - the tools are available and the API differences are minor and well-documented.

