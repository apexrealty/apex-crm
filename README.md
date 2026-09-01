# apex-crm

## GoHighLevel MCP server

The repo ships a project-scoped MCP config (`.mcp.json`) for
[`@drausal/gohighlevel-mcp`](https://www.npmjs.com/package/@drausal/gohighlevel-mcp),
which exposes the GoHighLevel v2 API (services.leadconnectorhq.com) as MCP tools.

### Setup

1. Create a Private Integration Token in GoHighLevel
   (Settings → Private Integrations), granting only the scopes you need.
2. Export it before starting Claude Code, so `.mcp.json` can interpolate it:

   ```bash
   export GHL_API_KEY=pit-xxxxxxxx
   ```

   Keep the token out of git — put it in a `.env` (gitignored) or your shell
   profile and source it. `.env.example` lists the variables.
3. Start Claude Code in this directory and approve the `ghl` server when
   prompted. Verify with `claude mcp list` or `/mcp`.

Requires Node.js >= 20.

### Notes on configuration

- **Auth variable names.** The server reads `BEARER_TOKEN_BEARERAUTH` and
  `BEARER_TOKEN_BEARER` (derived from the `bearerAuth` / `bearer` security
  schemes in its bundled OpenAPI spec). A variable named `GHL_API_KEY` passed
  straight to the server is ignored, and every API call then goes out
  unauthenticated. `.mcp.json` maps `GHL_API_KEY` onto both names.
- **Location ID.** The server has no location environment variable. Most of its
  tools take `locationId` as a call parameter, so keep your sub-account ID handy
  (`GHL_LOCATION_ID` in `.env.example`) and pass it per call.
- **Tool volume.** The server advertises 412 tools, roughly 140k tokens of
  schema. Expect that to dominate the context window, and disable the server
  when you are not actively using it.
- **Version pin.** `.mcp.json` pins `@1.0.0`. `npx -y @drausal/gohighlevel-mcp`
  without a version silently picks up new releases of a third-party package that
  holds a CRM token.

### Alternative: CLI setup instead of `.mcp.json`

```bash
claude mcp add --transport stdio \
  --env BEARER_TOKEN_BEARERAUTH="$GHL_API_KEY" \
  --env BEARER_TOKEN_BEARER="$GHL_API_KEY" \
  ghl -- npx -y @drausal/gohighlevel-mcp@1.0.0
```

This writes to your user-level config rather than the repo, so teammates don't
pick it up.
