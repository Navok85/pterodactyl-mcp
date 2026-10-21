# pterodactyl-mcp

MCP server for managing [Pterodactyl Panel](https://pterodactyl.io) servers. Supports both HTTP (Streamable HTTP) and stdio transports.

## Features

- **Admin tools** (application API) — manage users, servers, nodes, locations, nests, eggs, databases, and allocations
- **Client tools** (client API) — control servers, manage files, backups, schedules and tasks, subusers, databases, startup variables, network allocations, and account/API/SSH keys
- **WebSocket console** — real-time server console streaming via Pterodactyl's websocket API

## Setup

### 1. Install dependencies

```bash
npm install
```

### 2. Configure environment

Copy the example env file and fill in your panel credentials:

```bash
cp .env.example .env
```

| Variable | Description |
|---|---|
| `PTERODACTYL_URL` | Your panel URL (e.g. `https://panel.example.com`) |
| `PTERODACTYL_APP_KEY` | Application API key (`ptla_*`) for admin operations (optional) |
| `PTERODACTYL_CLIENT_KEY` | Client API key (`ptlc_*`) for client operations (optional) |
| `PORT` | HTTP server port (default: `3000`) |
| `MCP_JSON_LIMIT` | Maximum JSON request size for HTTP transport (default: `50mb`; increase for larger uploads) |
| `MCP_AUTH_TOKEN` | Optional bearer token required on `/mcp` (HTTP transport). Unset = no authentication |

At least one API key is required. You can use both to enable all tools.

## Uploading files and folders

The `upload_files` client tool uploads binary files to a server. Provide file bytes as standard base64 and use relative paths to preserve nested folders. Folder paths can also be supplied explicitly, which is useful for creating empty directories. The destination directory must already exist. Upload URLs are requested individually for each destination directory, as required by the panel's signed upload endpoint.

Example tool arguments:

```json
{
  "server_id": "abc123",
  "directory": "/",
  "files": [
    { "path": "plugins/example.jar", "content_base64": "<base64 file bytes>" },
    { "path": "config/settings.yml", "content_base64": "<base64 file bytes>" }
  ],
  "directories": ["empty-folder"]
}
```

Files are uploaded through Pterodactyl's temporary upload URL in groups by destination folder. Large uploads may require increasing `MCP_JSON_LIMIT`; the configured limit applies to HTTP transport requests.

For `write_file`, pass `content` as the plain text to write, with real newline characters (do not JSON-stringify it). To preserve exact file bytes and line endings, pass `content_base64` instead. If a client has already wrapped the text as a JSON string, set `content_json_encoded` to `true` to decode that string before writing.

### 3. Run

**Development:**

```bash
npm run dev
```

**Production:**

```bash
npm run build
npm start
```

## Transport modes

### HTTP (default)

Starts an Express server on the configured port:

```bash
npm start
# or
npx @duruma/pterodactyl-mcp
```

The MCP endpoint is available at `http://localhost:3000/mcp`. A health check is at `/health`.

If `MCP_AUTH_TOKEN` is set, every request to `/mcp` must send it as a bearer token:

```bash
curl -H "Authorization: Bearer $MCP_AUTH_TOKEN" http://localhost:3000/mcp
```

MCP clients must send the same header, e.g. `"headers": { "Authorization": "Bearer $MCP_AUTH_TOKEN" }`. `/health` stays unauthenticated, and stdio transport ignores this variable.

### Stdio

For use with MCP clients that communicate over stdin/stdout:

```bash
npx @duruma/pterodactyl-mcp --stdio
```

## MCP client configuration

Add to your MCP client config (e.g. `.mcp.json`, Claude Desktop, etc.):

```json
{
  "mcpServers": {
    "pterodactyl": {
      "command": "npx",
      "args": ["@duruma/pterodactyl-mcp", "--stdio"],
      "env": {
        "PTERODACTYL_URL": "https://panel.example.com",
        "PTERODACTYL_APP_KEY": "ptla_your_application_key_here",
        "PTERODACTYL_CLIENT_KEY": "ptlc_your_client_key_here"
      }
    }
  }
}
```

## License

[MIT](LICENSE)
