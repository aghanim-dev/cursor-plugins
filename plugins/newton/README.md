# Aghanim for Cursor

Search Aghanim's product documentation and browse the Aghanim public API directly from Cursor.

## What's included

- **Newton MCP server** — remote streamable HTTP connection to `https://mcp.aghanim.com/mcp`, authenticated with Aghanim SSO (Auth0).
- **`aghanim-webhooks-quick-start` skill** — walks Cursor through implementing Aghanim's `player.verify`, `item.add`, and `item.remove` webhook handlers in the current project, using the Newton MCP tools as the source of truth for docs and schemas.
- **`/aghanim-webhooks-quick-start` command** — runs the skill above on demand.

## Requirements

- An Aghanim account (SSO login).
- Cursor with plugin support.

## First-time use

When Cursor first calls an `mcp__aghanim__*` tool, a browser window opens for Aghanim SSO login. Complete it and return — the session persists.

## Support

Contact your Aghanim representative.

## License

Copyright (C) 2026 Aghanim, Inc — released under [AGPL-3.0-or-later](../../LICENSE).
