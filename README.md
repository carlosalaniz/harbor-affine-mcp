# AFFiNE MCP for Harbor

A [Harbor](https://github.com/carlosalaniz/harbor) app:
[affine-mcp-server](https://github.com/dawncr0w/affine-mcp-server) 3.8.0, an MCP gateway that lets
AI assistants (Claude, Cursor, any MCP client) read and edit an [AFFiNE](https://affine.pro)
workspace. Image pinned by digest. Needs Harbor 0.21.0 or later (operator-provided secrets and app
links) and the [AFFiNE package](https://github.com/carlosalaniz/harbor-affine) installed.

```
harbor/            the Harbor package
  manifest.yaml    endpoint (mcp, port 3000), health (/healthz), link to AFFiNE, values you provide
  compose.yaml     one service, affine-mcp
  README.md        shown in Harbor's App Store
```

## Install

In Harbor: **App Store → Your own app → Git repository** with this repository's HTTPS URL and
branch `main` (or `harbor sources add https://github.com/carlosalaniz/harbor-affine-mcp --branch main`),
then install **AFFiNE MCP**. The review dialog asks for three values:

| Value | What to type |
|---|---|
| `affine-email` | Email of an existing AFFiNE account; the gateway acts as this user |
| `affine-password` | That account's password |
| `mcp-http-token` | The bearer token MCP clients will send (16+ characters). Moving an existing gateway: type its current token so clients keep working. New: `openssl rand -hex 32` |

From the CLI (values from files or stdin, never on the command line):

```sh
harbor install affine-mcp --link affine=affine \
  --secret affine-email=@email.txt --secret affine-password=@password.txt \
  --secret mcp-http-token=@token.txt
```

Change any of them later on the app page (*Values you provided → Change…*) or with
`harbor configure <app> --secret mcp-http-token=@new.txt`.

## How it reaches AFFiNE

The `affine` link makes Harbor create a private network joined only by this gateway and AFFiNE's
web service (not its database or Redis), with no route to the internet. AFFiNE answers there as
`affine-link:3010`, and Harbor sets `AFFINE_BASE_URL=http://affine-link:3010`. Traffic on that
network is plain http, so `AFFINE_ALLOW_INSECURE_HTTP=true` is set. If AFFiNE is removed, the app is
flagged *needs a provider* until you link it again.

## Connecting an MCP client

Use Harbor's address for this app (its card / app page) plus `/mcp`, and the token you typed:

```json
{
  "mcpServers": {
    "affine": {
      "type": "http",
      "url": "https://<harbor address of AFFiNE MCP>/mcp",
      "headers": { "Authorization": "Bearer <mcp-http-token>" }
    }
  }
}
```

Claude Code: `claude mcp add --transport http affine https://<address>/mcp --header "Authorization: Bearer <token>"`.
The legacy SSE transport is also served at `/sse`. Send the token in the header, never in the URL.

| Path | Auth | Answers |
|---|---|---|
| `/mcp` | bearer | Streamable HTTP MCP |
| `/sse` | bearer | Legacy SSE MCP |
| `/healthz` | none | The gateway is up (Harbor's health check) |
| `/readyz` | none | The gateway reaches AFFiNE's GraphQL API (503 otherwise) |

## Settings in the package

`MCP_TRANSPORT=http`, `AFFINE_MCP_AUTH_MODE=bearer`, `AFFINE_TOOL_PROFILE=full` (every tool,
including destructive ones), listening on `0.0.0.0:3000`. Nothing secret lives in this repository.

affine-mcp-server is MIT-licensed by its contributors; AFFiNE is © Toeverything Pte. Ltd. This
repository only contains the packaging and pulls the upstream image.
