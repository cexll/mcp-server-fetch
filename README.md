# Fetch MCP Server

A Model Context Protocol server that provides web content fetching capabilities. This server enables LLMs to retrieve and process content from web pages, converting HTML to markdown for easier consumption.

**This fork disables robots.txt checking by default**, allowing unrestricted domain fetching. You can enable robots.txt compliance by adding the `--respect-robots-txt` flag.

The fetch tool will truncate the response, but by using the `start_index` argument, you can specify where to start the content extraction. This lets models read a webpage in chunks, until they find the information they need.

### Available Tools

- `fetch` - Fetches a URL from the internet and extracts its contents as markdown.
    - `url` (string, required): URL to fetch
    - `max_length` (integer, optional): Maximum number of characters to return (default: 5000)
    - `start_index` (integer, optional): Start content from this character index (default: 0)
    - `raw` (boolean, optional): Get raw content without markdown conversion (default: false)

### Prompts

- **fetch**
  - Fetch a URL and extract its contents as markdown
  - Arguments:
    - `url` (string, required): URL to fetch

## Installation

Optionally: Install node.js, this will cause the fetch server to use a different HTML simplifier that is more robust.

### Using uv (recommended)

When using [`uv`](https://docs.astral.sh/uv/) no specific installation is needed. We will
use [`uvx`](https://docs.astral.sh/uv/guides/tools/) to directly run *mcp-server-fetch*.

### Using PIP

Alternatively you can install `mcp-server-fetch` via pip:

```
pip install mcp-server-fetch
```

After installation, you can run it as a script using:

```
python -m mcp_server_fetch
```

## Configuration

### Quick Setup (Recommended)

Use the Claude CLI to add this server globally:

```bash
claude mcp add-json --scope user fetch '{
  "command": "uvx",
  "args": [
    "--from",
    "git+https://github.com/cexll/mcp-server-fetch.git",
    "mcp-server-fetch"
  ]
}'
```

Verify installation:
```bash
claude mcp list
```

### Configure for Claude.app

Add to your Claude settings:

<details>
<summary>Using uvx</summary>

```json
"mcpServers": {
  "fetch": {
    "command": "uvx",
    "args": ["mcp-server-fetch"]
  }
}
```
</details>

<details>
<summary>Using docker</summary>

```json
"mcpServers": {
  "fetch": {
    "command": "docker",
    "args": ["run", "-i", "--rm", "mcp/fetch"]
  }
}
```
</details>

<details>
<summary>Using pip installation</summary>

```json
"mcpServers": {
  "fetch": {
    "command": "python",
    "args": ["-m", "mcp_server_fetch"]
  }
}
```
</details>

### Customization - robots.txt

**This fork ignores robots.txt by default**, allowing unrestricted fetching of any domain. If you want to respect robots.txt restrictions, add the `--respect-robots-txt` flag to the `args` list in the configuration.

Example:
```bash
claude mcp add-json --scope user fetch '{
  "command": "uvx",
  "args": [
    "--from",
    "git+https://github.com/cexll/mcp-server-fetch.git",
    "mcp-server-fetch",
    "--respect-robots-txt"
  ]
}'
```

### Customization - User-agent

By default, depending on if the request came from the model (via a tool), or was user initiated (via a prompt), the
server will use either the user-agent
```
ModelContextProtocol/1.0 (Autonomous; +https://github.com/modelcontextprotocol/servers)
```
or
```
ModelContextProtocol/1.0 (User-Specified; +https://github.com/modelcontextprotocol/servers)
```

This can be customized by adding the argument `--user-agent=YourUserAgent` to the `args` list in the configuration.

Example:
```bash
claude mcp add-json --scope user fetch '{
  "command": "uvx",
  "args": [
    "--from",
    "git+https://github.com/cexll/mcp-server-fetch.git",
    "mcp-server-fetch",
    "--user-agent=MyBot/1.0"
  ]
}'
```

### Customization - Proxy

The server can be configured to use a proxy by using the `--proxy-url` argument.

Example:
```bash
claude mcp add-json --scope user fetch '{
  "command": "uvx",
  "args": [
    "--from",
    "git+https://github.com/cexll/mcp-server-fetch.git",
    "mcp-server-fetch",
    "--proxy-url=http://proxy.example.com:8080"
  ]
}'
```

## Debugging

You can use the MCP inspector to debug the server. For uvx installations:

```
npx @modelcontextprotocol/inspector uvx mcp-server-fetch
```

Or if you've installed the package in a specific directory or are developing on it:

```
cd path/to/servers/src/fetch
npx @modelcontextprotocol/inspector uv run mcp-server-fetch
```

## Contributing

We encourage contributions to help expand and improve mcp-server-fetch. Whether you want to add new tools, enhance existing functionality, or improve documentation, your input is valuable.

This is a fork of the official MCP fetch server with robots.txt checking disabled by default.

- **This fork**: https://github.com/cexll/mcp-server-fetch
- **Upstream**: https://github.com/modelcontextprotocol/servers/tree/main/src/fetch

Pull requests are welcome! Feel free to contribute new ideas, bug fixes, or enhancements to make mcp-server-fetch even more powerful and useful.

## License

mcp-server-fetch is licensed under the MIT License. This means you are free to use, modify, and distribute the software, subject to the terms and conditions of the MIT License. For more details, please see the LICENSE file in the project repository.
