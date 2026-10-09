# TRMNL

Design and run your TRMNL ePaper screens with Cursor.

This plugin gives Cursor two things:

- **The TRMNL skill.** The full TRMNL design system (layouts, framework classes, chart patterns and the v3 color palette) plus the rules TRMNL's own AI assistant follows, so the markup Cursor writes looks right on a real display.
- **The TRMNL MCP server** at `https://trmnl.com/mcp`, so Cursor can work on your account: devices, playlists, plugins, recipes and the markup of your private plugins.

## Try asking

- "Build me a half-size weather screen that fits the TRMNL OG and the TRMNL X."
- "Why does my private plugin look cramped on the OG?"
- "Add my calendar to the kitchen display on weekday mornings."

## What it connects to

The skill is plain text and runs nothing on your machine. The plugin adds one remote MCP server, `https://trmnl.com/mcp`, run by TRMNL. The first time Cursor uses it, you sign in with your TRMNL account (OAuth) and choose what Cursor may do. Deleting starts off. Tool calls go only to trmnl.com, and nothing is downloaded or installed. You can see or revoke the connection any time under Account, then Developer, then Connected agents.

- Setup guide: https://help.trmnl.com/en/articles/17432548-mcp-server
- Privacy policy: https://trmnl.com/privacy

## License

MIT
