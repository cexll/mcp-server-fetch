# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server that provides web content fetching capabilities to LLMs. It fetches web pages, extracts content, and converts HTML to markdown format for easier consumption by language models.

## Development Commands

### Running the Server

```bash
# Run with uv (recommended)
uv run mcp-server-fetch

# Run with Python module
python -m mcp_server_fetch
```

### Debugging

```bash
# Use MCP inspector for debugging
npx @modelcontextprotocol/inspector uv run mcp-server-fetch
```

### Code Quality

```bash
# Run type checking (using pyright from dev-dependencies)
uv run pyright

# Run linting (using ruff from dev-dependencies)
uv run ruff check
```

## Architecture

### Core Components

**server.py:181-288** - Main server implementation
- `serve()`: Async entry point that creates and runs the MCP server
- Implements three MCP endpoints: `list_tools()`, `call_tool()`, and `get_prompt()`
- Handles both autonomous (tool-based) and manual (prompt-based) fetch requests with different user-agents

**Fetch Flow**:
1. Tool/prompt invoked with URL and optional parameters
2. For autonomous requests: check robots.txt compliance via `check_may_autonomously_fetch_url()`
3. Fetch URL via `fetch_url()` using httpx AsyncClient
4. Detect content type (HTML vs other)
5. For HTML: extract and convert to markdown via `extract_content_from_html()`
6. Apply pagination: truncate to max_length starting from start_index
7. Return content with truncation notice if needed

### Key Design Patterns

**Dual User-Agent Strategy** (server.py:23-24, 194-195):
- Autonomous requests (via tool): `ModelContextProtocol/1.0 (Autonomous; ...)`
- Manual requests (via prompt): `ModelContextProtocol/1.0 (User-Specified; ...)`
- This distinction allows websites to differentiate between AI-initiated vs user-initiated requests

**Robots.txt Compliance** (server.py:66-108):
- Only enforced for autonomous (tool) requests, not manual (prompt) requests
- Fetches and parses robots.txt using protego library
- Returns detailed error messages including the robots.txt content when blocked
- Can be disabled via `--ignore-robots-txt` flag

**Content Pagination** (server.py:240-254):
- Uses `start_index` to allow chunked reading of long pages
- Returns truncation notice with next start_index when content exceeds max_length
- Enables models to incrementally fetch content until they find needed information

**HTML Simplification** (server.py:27-45):
- Uses readabilipy to extract main content (removes boilerplate)
- Converts to markdown using markdownify for LLM-friendly format
- Falls back to raw content if simplification fails or for non-HTML content

### Configuration Options

All passed as command-line arguments:
- `--user-agent`: Override default user-agent strings
- `--ignore-robots-txt`: Bypass robots.txt checking for autonomous requests
- `--proxy-url`: Route requests through specified proxy

### Dependencies

Critical dependencies (see pyproject.toml:18-26):
- `httpx<0.28`: HTTP client (pinned due to proxy compatibility issue)
- `mcp>=1.1.3`: MCP protocol SDK
- `readabilipy>=0.2.0`: HTML content extraction
- `markdownify>=0.13.1`: HTML to markdown conversion
- `protego>=0.3.1`: robots.txt parsing

### Entry Points

- `__init__.py:4-21`: Main entry with argparse setup
- `__main__.py`: Module entry point (delegates to main())
- `pyproject.toml:28-29`: Defines `mcp-server-fetch` console script

## Common Development Scenarios

### Adding a new fetch parameter

1. Update `Fetch` Pydantic model in server.py:151-178
2. Pass parameter through `fetch_url()` function
3. Update tool description in `list_tools()` to document new parameter

### Modifying HTML extraction logic

- Edit `extract_content_from_html()` in server.py:27-45
- This function controls how HTML is simplified and converted to markdown

### Adding proxy support improvements

- Main proxy logic is in `fetch_url()` (server.py:111-148) and `check_may_autonomously_fetch_url()` (server.py:66-108)
- Both use httpx AsyncClient with `proxies=proxy_url` parameter
- Note: httpx pinned to <0.28 due to proxy compatibility issues (see commit 9a4d513)

## Testing

To test the server manually:
1. Run with MCP inspector: `npx @modelcontextprotocol/inspector uv run mcp-server-fetch`
2. Use the inspector UI to invoke tools and prompts
3. Test with various URLs, including those with robots.txt restrictions
