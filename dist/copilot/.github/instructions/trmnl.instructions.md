---
applyTo: "**"
description: Use when working on anything TRMNL — building plugins, writing or editing
  markup, designing layouts using the TRMNL CSS framework, calling the TRMNL MCP server
  (mcp__trmnl__*), or any task that mentions "trmnl", "e-ink display", "trmnl.com",
  "TRMNL plugin", "Liquid markup", "image-dither", "title_bar", or screen sizes 800x480
  / 800x240 / 400x480 / 400x240. References embed the production prompts TRMNL's external
  MCP server ships — agent rules, the full design system template guide, and the framework
  v3 supplement.
---

# TRMNL

Everything you need to work on TRMNL. The references in this skill are **verbatim copies** of the prompts TRMNL's own production AI assistant reads.

## Read first, in this order

| # | Reference | What it covers | Lines |
|---|---|---|---|
| 1 | [`references/agent_prompt.md`](../refs/agent_prompt.md) | Core rules: data-first hard gate, mandatory workflows, view dimensions, layout system, image dithering, no-custom-styles, no-emojis, common mistakes. **This is also what TRMNL's MCP server ships as the connection-level instructions to every external client.** | 427 |
| 2 | [`references/template_guide.md`](../refs/template_guide.md) | THE design system reference — every framework class, layout pattern, chart code, item component, custom-fields YAML schema. Bundled here so non-MCP users can read it offline; MCP-connected users can also fetch it via the design-system template-guide tool. | 2710 |
| 3 | [`references/framework_v3_guide.md`](../refs/framework_v3_guide.md) | v3 supplement: chromatic palette, CSS variables, label variants. v3.0.3+ specific. | 230 |

## Adding the TRMNL MCP server (optional)

The skill works standalone — agents read the bundled references to write TRMNL-compliant markup locally without ever calling MCP. Adding TRMNL's hosted MCP server unlocks **live plugin operations**: reading/writing markup on your real plugins, screenshots, merge variable inspection, recipe search.

1. **Get your API key:** TRMNL dashboard → any plugin → settings → MCP tab.
2. **Register the server with your agent:**

   | Agent | Setup |
   |---|---|
   | Claude Code | `claude mcp add --transport http trmnl "https://trmnl.com/mcp?api_key=<api-key>"` |
   | Cursor | Edit `~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (per-project) and add: `{"mcpServers": {"trmnl": {"url": "https://trmnl.com/mcp?api_key=<api-key>", "type": "http"}}}` — restart Cursor afterward. |
   | Codex, Gemini, generic | Add to your MCP config JSON: `{"mcpServers": {"trmnl": {"url": "https://trmnl.com/mcp?api_key=<api-key>"}}}` |

3. **Verify:** ask your agent to list TRMNL MCP tools — you should see `MarkupsReadTool`, `MarkupsWriteTool`, `MarkupsScreenshotTool`, etc.

**Endpoint:** `POST https://trmnl.com/mcp?api_key=<api-key>`. Rate limit: 60 req / 60s.

## Tool name mapping

The references reference TRMNL tools two different ways depending on context. Map between them:

| In-app agent (snake_case) | MCP server (PascalCase + Tool) |
|---|---|
| `show_integration` | `IntegrationsShowTool` |
| `show_merge_variables` | `MergeVariablesShowTool` |
| `write_settings` | `IntegrationsWriteSettingsTool` |
| `show_logs` | `IntegrationsLogsTool` |
| `refresh_data` | `IntegrationsRefreshDataTool` (async — dispatch via `AsyncStartTool`) |
| `read_markup` | `MarkupsReadTool` |
| `write_markup` | `MarkupsWriteTool` |
| `list_markup_sizes` | `MarkupsListSizesTool` |
| `screenshot_markup` | `MarkupsScreenshotTool` (async — dispatch via `AsyncStartTool`) |
| `pull_recipe_markup` | `RecipesPullMarkupTool` |
| (no in-app equivalent) | `RecipesSearchTool`, `DesignSystemReferenceTool`, `DesignSystemTemplateGuideTool`, `APIEndpointsSearchTool`, `AsyncStartTool`, `AsyncResultTool` |

If you're calling MCP from outside TRMNL's web app, use the right column. The references use the left column — translate as you go.

## What's in-app-only (translate or ignore)

`agent_prompt.md` was written for TRMNL's *in-app* AI assistant (the web chat at trmnl.com). A handful of sections are about that web UI specifically and don't apply to external agents (Claude Code, Cursor, Codex, Gemini, Copilot). Translate as you read:

| Reference | What it says | What to do externally |
|---|---|---|
| `agent_prompt.md:35` | "NEVER write raw HTML/markup in your chat response" | Applies only to TRMNL's in-app chat (where markup interferes with the live editor). External agents — show markup in chat freely; users often want to review it before committing. |
| `agent_prompt.md:71, 208` | "use ask_user" | `ask_user` is an in-app tool. Externally: just ask the user a normal question. |
| `agent_prompt.md:73` | "the Self-Correction Workflow (loaded separately in this system prompt)" | The mandate to screenshot after every write still applies. Iterate; react to overflow signals in the response; look for washed images (missing `image-dither`) and empty boxes (emoji used). |
| `agent_prompt.md:95` | `write_markup` "broadcasts live update to browser editor" | Web UI artifact. `MarkupsWriteTool` writes to the database; there's no editor to update. Ignore the broadcast wording. |
| `agent_prompt.md:99-102` | Tools listed: `preview_markup`, `validate_liquid`, `version_history`, `ask_user` | None of these exist as external MCP tools. Skip workflow steps that depend on them. |

Everything else in `agent_prompt.md` applies universally — design rules, e-ink constraints, image dithering, no-custom-styles, no-emojis, the data-first hard gate, spatial proportioning, layout system, charts.

`template_guide.md` and `framework_v3_guide.md` are pure design system reference — no in-app artifacts.

## NOT for

- The TRMNL **REST API** for outside-MCP integrations — see [the OpenAPI spec at `/api-docs`](https://trmnl.com/api-docs) directly.
- TRMNL **firmware** — different repo.
- Generic Liquid/Shopify/Jekyll questions — TRMNL Liquid has TRMNL-specific filters and merge variables.

## How to use the references

These files are LARGE. Read selectively:

- Starting any markup work? → `agent_prompt.md` (mandatory workflows section), then jump to relevant `template_guide.md` section by topic.
- Picked a v3 plugin? → also load `framework_v3_guide.md` (color rules differ from v2).
- Need a specific framework class or layout pattern? → search `template_guide.md` directly.

Don't load all 3 files at once unless you have to. Each is independently useful.
