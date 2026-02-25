# Task 03: Web Tools Verification - Final QA

**Date**: 2026-02-25
**Scenario**: Verify web tools
**Status**: ✅ PASS

## QA Scenario
```
Scenario: Verify web tools
  Tool: Bash
  Steps:
    1. Check if websearch tool exists
    2. Check if webfetch tool exists
  Expected Result: API documentation created
  Evidence: .sisyphus/evidence/task-03-web-tools-api.md
```

## Execution Results

### Step 1: Check if websearch tool exists
- **Tool Available**: ✅ Yes (`websearch_web_search_exa`)
- **Type**: Built-in SDK tool
- **Location**: `~/.opencode/node_modules/@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts`

**API Signature**:
```typescript
{
  query: string;                    // Search query (required)
  numResults?: number;              // Number of results (default: 8, must be a number)
  type?: "auto" | "fast";           // Search type (default: "auto")
  contextMaxCharacters?: number;   // Max characters for context (default: 10000)
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

### Step 2: Check if webfetch tool exists
- **Tool Available**: ✅ Yes (`webfetch`)
- **Type**: Built-in SDK tool
- **Location**: `~/.opencode/node_modules/@opencode-ai/sdk/dist/v2/gen/types.gen.d.ts`

**API Signature**:
```typescript
{
  url: string;                              // Fully-formed valid URL (required)
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

### Permission Configuration
Both tools are configured in the SDK permission system:
```typescript
export type AgentPermissionsConfig = {
    webfetch?: PermissionActionConfig;   // Permission config for webfetch
    websearch?: PermissionActionConfig;  // Permission config for websearch
}
```

**Permission Actions**: "ask" | "allow" | "deny"

## Verification Checklist

- [x] websearch_web_search_exa tool exists as built-in SDK tool
- [x] webfetch tool exists as built-in SDK tool
- [x] API syntax documented for both tools
- [x] Tool permission configuration confirmed
- [x] No MCP configuration required (built-in SDK tools)
- [x] API documentation created

## Comparison: Claude Code vs OpenCode

| Feature | Claude Code | OpenCode | Migration Status |
|---------|-------------|----------|-----------------|
| Web Search Tool | `WebSearch` | `websearch_web_search_exa` | ✅ Compatible |
| Web Fetch Tool | `WebFetch` | `webfetch` | ✅ Compatible |
| Search Query | query string | `query` parameter | ✅ Same |
| Number of Results | Configurable | `numResults` parameter | ✅ Same |
| Output Format | Auto-detect | `format` parameter (text/markdown/html) | ✅ Enhanced |
| Timeout | Configurable | `timeout` parameter (max 120s) | ✅ Same |
| Live Crawl | Not documented | `livecrawl` parameter (fallback/preferred) | ✅ Enhanced |
| Built-in vs MCP | Built-in | Built-in SDK | ✅ Same |

## Conclusion

**Task 3 QA Result**: ✅ PASS

OpenCode provides both `websearch_web_search_exa` and `webfetch` as built-in SDK tools with functionality equivalent to or better than Claude Code's WebSearch and WebFetch. No blocker exists for the dr-ralph migration - the tools are available and the API differences are minor and well-documented.

**Key Finding**: Both tools are built-in SDK tools, not MCP-based, meaning they're available out-of-the-box in OpenCode environment without additional configuration.

**Evidence file exists**: `.sisyphus/evidence/task-03-web-tools-api.md`
