# UseMo MCP

UseMo brings video, image, and branded-content creation into MCP clients. It
connects to your own UseMo workspace through OAuth; generation uses your
workspace's existing credits.

The remote server is available at `https://api.usemo.com/mcp` using Streamable
HTTP. It provides tools to plan and quote content, create videos and images,
generate written collateral, check jobs, and retrieve finished assets.

## Cursor

Install this repository as a Cursor plugin, or add this entry to your Cursor MCP
configuration:

```json
{
  "mcpServers": {
    "usemo": {
      "url": "https://api.usemo.com/mcp"
    }
  }
}
```

Cursor will open the UseMo OAuth flow when a tool needs your account.

## Claude

In Claude, open **Customize → Connectors → Add custom connector**, enter
`https://api.usemo.com/mcp`, and complete OAuth. This remote connector works
before any directory listing decision.

## Local stdio MCP

For MCP clients that prefer local processes, install the published SDK:

```bash
npm install -g @usemo.com/sdk@0.1.0
printf '%s\\n' "$USEMO_API_KEY" | usemo login --key-stdin --profile work
```

Then configure:

```json
{
  "mcpServers": {
    "usemo": {
      "command": "npx",
      "args": ["--yes", "--package", "@usemo.com/sdk@0.1.0", "usemo-mcp"],
      "env": { "USEMO_PROFILE": "work" }
    }
  }
}
```

Requires Node.js 20 or newer. Do not commit API keys into configuration files.

## Credit and job behavior

Generation returns a job handle. Wait for or fetch the job before retrying: a
client timeout does not cancel work. If a workspace is out of credits, UseMo
returns an actionable insufficient-credits error and does not start duplicate
work or add credits automatically.

## Support

- Website: https://usemo.com
- Privacy: https://usemo.com/privacy
- Terms: https://usemo.com/terms
- Support: support@usemo.com

## License

MIT. This repository contains integration configuration and documentation only;
the UseMo hosted service and its APIs are governed by the UseMo Terms of Service.
