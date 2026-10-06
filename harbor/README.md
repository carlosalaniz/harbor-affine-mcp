# AFFiNE MCP

An MCP gateway for AFFiNE (affine-mcp-server 3.8.0): AI assistants read and edit your AFFiNE
workspaces through it, signed in as one AFFiNE account. Install the AFFiNE app first.

## At install

Harbor links this app privately to AFFiNE (only AFFiNE's web service joins; it answers as
`affine-link:3010`) and asks for:

- **affine-email** and **affine-password**: an existing AFFiNE account the gateway acts as.
- **mcp-http-token**: the bearer token MCP clients send (16+ characters). Keep your current token
  when moving an existing gateway; otherwise use a long random one (`openssl rand -hex 32`).

Change them later under *Values you provided → Change…*.

## Connecting a client

This is not a web page. Point an MCP client at this app's address plus `/mcp` with the header
`Authorization: Bearer` and the token (legacy SSE: `/sse`). `/healthz` answers without the token
(Harbor's health check); `/readyz` reports whether the gateway reaches AFFiNE.

Tool profile: `full` (every tool, including ones that delete or replace documents).
