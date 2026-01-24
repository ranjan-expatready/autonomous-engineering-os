# Arabold Docs MCP Server Installation Artifact

## Overview

This artifact documents the installation and configuration of the arabold Docs MCP Server (`docs_arabold`) as an additional documentation intelligence capability for the Autonomous Engineering OS.

---

## Updated MCP Server List

The Factory MCP configuration now includes the following servers:

### 1. context7
- **Type**: HTTP
- **URL**: `https://mcp.context7.com/mcp`
- **Purpose**: Remote context provider for general knowledge
- **Status**: Enabled

### 2. chrome-devtools
- **Type**: Stdio
- **Command**: `npx -y chrome-devtools-mcp@latest`
- **Purpose**: Chrome DevTools browser automation
- **Status**: Enabled

### 3. playwright
- **Type**: Stdio
- **Command**: `npx -y @playwright/mcp@latest`
- **Purpose**: Playwright browser automation and testing
- **Status**: Enabled

### 4. filesystem
- **Type**: Stdio
- **Command**: `npx @modelcontextprotocol/server-filesystem`
- **Scope**: `/Users/ranjansingh/Desktop/autonomous-engineering-os`
- **Purpose**: Repository filesystem access
- **Status**: Enabled

### 5. docs (Original - Unchanged)
- **Type**: HTTP
- **URL**: `https://modelcontextprotocol.io/mcp`
- **Purpose**: Official MCP protocol documentation
- **Status**: Enabled

### 6. docs_arabold (NEW)
- **Type**: Stdio
- **Command**: `npx -y docs-mcp-server mcp --protocol stdio`
- **Purpose**: Local documentation indexing and search for libraries/frameworks
- **Status**: Enabled

---

## What docs_arabold Adds

### 5-Bullet Explanation

1. **Local Documentation Indexing**: Provides semantic search across indexed documentation for thousands of libraries, frameworks, and APIs. Content is indexed locally, enabling fast, offline-capable queries without external network calls.

2. **Version-Specific API Knowledge**: Automatically resolves and serves documentation for specific library versions (e.g., React 18.3.1, Django 5.0). Ensures agents reference accurate API signatures that match the version being used in the project.

3. **Structured Query Interface**: Exposes MCP tools for semantic search, version resolution (`find-version`), documentation listing (`list`), and URL fetching (`fetch-url`). Agents can query specific aspects of documentation (e.g., "useState hook usage examples").

4. **Built-in Caching and Refresh**: Maintains a local index that automatically refreshes documentation on-demand. Supports manual refresh commands (`refresh`) to ensure docs stay current with upstream changes.

5. **Enhanced Technical Accuracy**: Reduces hallucination risk by grounding all technical queries in actual documentation. Agents no longer rely on memory or training data for versioned APIs, which frequently change across releases.

---

## When to Use docs_arabold

### Recommended Use Cases

**Use docs_arabold when:**
- Querying specific framework/library API details (e.g., "How does React 19's use() hook work?")
- Looking for code examples from official documentation
- Verifying function signatures, parameters, and return types
- Searching for best practices from authoritative sources
- Needing version-specific documentation (e.g., Django 4.2 vs 5.0)
- Finding migration guides or breaking changes between versions

**Do NOT use docs_arabold when:**
- Looking up project-specific patterns (use Repo Doctrine: `FRAMEWORK_KNOWLEDGE/`)
- Querying MCP protocol specifics (use `docs` MCP server - official documentation)
- Searching for very recent announcements or news (use WebSearch)
- Retrieving general knowledge without technical specifics (use context7)

---

## Priority Order for Documentation Sources

1. **Repo Doctrine** (HIGHEST) → Project-specific patterns and decisions
2. **docs MCP** → Official MCP protocol documentation
3. **docs_arabold MCP** (NEW) → Indexed library/framework documentation
4. **context7 MCP** → Remote context knowledge
5. **WebSearch/FetchUrl** (LOWEST) → General web search

---

## Governance References

### Updated Documents

**1. GOVERNANCE/GUARDRAILS.md**
- Added "DOCUMENTATION SOURCES POLICY" section
- Defines priority order for documentation sources
- Provides decision matrix for choosing appropriate source
- Includes versioned API usage guidelines

**2. AGENTS/BEST_PRACTICES.md**
- Added Best Practice #26: "Use Docs MCP for Technical Facts"
- Mandates use of docs/docs_arabold for versioned APIs
- Prohibits reliance on memory for API signatures
- Includes example BAD vs GOOD documentation citation

---

## Configuration File

**Location**: `/Users/ranjansingh/.factory/mcp.json`

**docs_arabold Configuration**:
```json
"docs_arabold": {
  "type": "stdio",
  "command": "npx",
  "args": [
    "-y",
    "docs-mcp-server",
    "mcp",
    "--protocol",
    "stdio"
  ],
  "note": "Arabold Docs MCP Server - Local documentation indexing and search",
  "disabled": false
}
```

---

## Installation Details

### Version Installed
- **Package**: `@arabold/docs-mcp-server`
- **Install Method**: Local project install (within repo)
- **Cache Workaround**: Used custom cache directory to avoid permission issues
- **Verification**: Confirmed working with `docs-mcp-server --help`

### Available Commands (for reference)
```
Usage: docs-mcp-server <command> [options]

Commands:
  docs-mcp-server config        Fetch a URL and transform it into Markdown format
  docs-mcp-server server        Start the Docs MCP server (Unified Mode) [default]
  docs-mcp-server fetch-url     Fetch a URL and transform it into Markdown format
  docs-mcp-server find-version  Resolve best matching documentation version
  docs-mcp-server list          List all indexed libraries and versions
  docs-mcp-server mcp           Start the MCP server (Standalone Mode)
  docs-mcp-server refresh       Update existing library version
  docs-mcp-server remove        Delete library documentation from index
  docs-mcp-server scrape        Download and index documentation from URL
  docs-mcp-server search        Query the documentation index
  docs-mcp-server web           Start the web dashboard (Standalone Mode)
  docs-mcp-server worker        Start background worker for scraping jobs
```

---

## Testing & Verification

To verify docs_arabold is working:

1. Restart Factory to load the new MCP server configuration
2. Test querying a specific library (e.g., "Search React useState documentation")
3. Verify version-specific queries work (e.g., "Find React 18.3.1 useEffect signature")
4. Confirm that MCP tools are exposed: `search`, `find-version`, `list`, `fetch-url`

---

## Impact on Existing Functionality

**No Breaking Changes**:
- All existing MCP servers remain unchanged (context7, chrome-devtools, playwright, filesystem, docs)
- Existing workflows continue to operate normally
- docs_arabold is an additional capability, not a replacement

**Enhanced Capabilities**:
- Agents now have access to versioned technical documentation
- Reduced reliance on memory and training data for APIs
- More accurate and authoritative technical citations

---

## Summary

The arabold docs MCP server (`docs_arabold`) adds robust, versioned technical documentation capabilities to the Autonomous Engineering OS. It serves as the preferred source for library/framework API queries, sitting in the documentation source hierarchy after repo doctrine and official MCP docs. Configuration is complete, governance is updated, and no existing functionality was modified.

---

## Installation Date
**2026-01-22**

## Installed By
**Factory CTO Agent (Droid)**

## Status
**✅ CONFIGURATION COMPLETE**
