# TRMNL Agent Skills

Auto-generated for the Gemini CLI from https://github.com/usetrmnl/trmnl-agent-skills.

Reference files are bundled in `refs/` next to this file. Read them as needed.

Everything you need to work on TRMNL. The references in this skill are **verbatim copies** of the prompts TRMNL's own production AI assistant reads.

## Read first, in this order

| # | Reference | What it covers | Lines |
|---|---|---|---|
| 1 | [`references/agent_prompt.md`](refs/agent_prompt.md) | Core rules: data-first hard gate, mandatory workflows, view dimensions, layout system, image dithering, no-custom-styles, no-emojis, common mistakes. **This is also what TRMNL's MCP server ships as the connection-level instructions to every external client.** | 427 |
| 2 | [`references/template_guide.md`](refs/template_guide.md) | THE design system reference — every framework class, layout pattern, chart code, item component, custom-fields YAML schema. Bundled here so non-MCP users can read it offline; MCP-connected users can also fetch it via the design-system template-guide tool. | 2710 |
| 3 | [`references/framework_v3_guide.md`](refs/framework_v3_guide.md) | v3 supplement: chromatic palette, CSS variables, label variants. v3.0.3+ specific. | 230 |

## Adding the TRMNL MCP server (optional)

The skill works standalone — agents read the bundled references to write TRMNL-compliant markup locally without ever calling MCP. Adding TRMNL's hosted MCP server unlocks **live operations** on your real account. One endpoint, `https://trmnl.com/mcp`, two ways in — the credential decides which tools you get:

| Connection | Credential | Tools you get |
|---|---|---|
| **One plugin** | MCP key from TRMNL dashboard → that plugin → settings → MCP tab, as `?api_key=` | The markup tools for that one plugin: read/write markup, screenshots, merge variables, logs, refresh, recipe search, design system reference |
| **Whole account** | Sign in with OAuth (no key). The account API key from <https://trmnl.com/account> is for the REST API only and answers 401 on `/mcp` | The seven account tools: devices, playlists and mashups, plugin settings, markup, profile, recipes, third-party API search — every action is an operation of the REST API |

The full authentication reference is <https://trmnl.com/auth.md>.

### Register the server with your agent

| Agent | One plugin (key) | Whole account (OAuth) |
|---|---|---|
| Claude Code | `claude mcp add --transport http trmnl "https://trmnl.com/mcp?api_key=<api-key>"` | `claude mcp add --transport http trmnl https://trmnl.com/mcp`, then `/mcp` in a session → sign in. A browser tab opens for consent and bounces to `http://localhost:<port>/callback`; that tab going blank afterward is normal, the CLI already took the code. |
| Cursor | Edit `~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (per-project) and add: `{"mcpServers": {"trmnl": {"url": "https://trmnl.com/mcp?api_key=<api-key>", "type": "http"}}}` — restart Cursor afterward. | Same entry without `?api_key=`; Cursor prompts for the OAuth sign-in. |
| Codex, Gemini, generic | Add to your MCP config JSON: `{"mcpServers": {"trmnl": {"url": "https://trmnl.com/mcp?api_key=<api-key>"}}}` | Same entry without `?api_key=`; the client must speak MCP OAuth (RFC 7591 registration, PKCE). |

**Verify:** ask your agent to list TRMNL MCP tools. One plugin: `MarkupsReadTool`, `MarkupsWriteTool`, `MarkupsScreenshotTool`, etc. Whole account: `AccountDevicesTool`, `AccountPlaylistsTool`, `AccountPluginSettingsTool`, `AccountMarkupTool`, `AccountProfileTool`, `AccountRecipesTool`, `APIEndpointsSearchTool`.

**Endpoint:** `POST https://trmnl.com/mcp`. Rate limit: 60 req / 60s. OAuth scopes are capabilities: `read` (list and read), `content` (markup, plugin data and fields, playlists, creating plugin settings), `devices` (device settings, identify), `delete` (plugin settings, playlist items), `profile` (`getMe`). Ask for all five; the user ticks what they want at consent (`delete` and `profile` start unticked) and may limit the connection to some devices and plugin settings. `write` is the older name for `content` + `devices` + `delete`.

## Tool name mapping

The references reference TRMNL tools two different ways depending on context. Map between them:

| In-app agent (snake_case) | MCP server (PascalCase + Tool) |
|---|---|
| `show_integration` | `IntegrationsShowTool` |
| `show_merge_variables` | `MergeVariablesShowTool` |
| `write_settings` | `IntegrationsWriteSettingsTool` |
| `show_logs` | `IntegrationsLogsTool` |
| `refresh_data` | `IntegrationsRefreshDataTool` (answers within 15s; for a slower fetch dispatch via `AsyncStartTool`) |
| `read_markup` | `MarkupsReadTool` |
| `write_markup` | `MarkupsWriteTool` |
| `list_markup_sizes` | `MarkupsListSizesTool` |
| `screenshot_markup` | `MarkupsScreenshotTool` (takes a `views` array and optional `device_models`; long renders answer a `job_id` to poll with `AsyncResultTool`) |
| `pull_recipe_markup` | `RecipesPullMarkupTool` |
| (no in-app equivalent) | `RecipesSearchTool`, `DesignSystemReferenceTool`, `DesignSystemTemplateGuideTool`, `APIEndpointsSearchTool`, `AsyncStartTool`, `AsyncResultTool` |

If you're calling MCP from outside TRMNL's web app, use the right column. The references use the left column — translate as you go.

### Account tools (whole-account connection)

Each account tool takes an `action` (the REST API operation) and a `params` hash. The tool description lists every action with its parameters, accepted values and shapes — read it before guessing a shape.

| Tool | Actions |
|---|---|
| `AccountProfileTool` | `getMe`, `updateMe`, `listModels`, `listPalettes`, `listCategories` |
| `AccountDevicesTool` | `listDevices`, `getDevice`, `updateDevice`, `identifyDevice`, `getDeviceLogs`, `clearDevicePlaylist` (destructive) |
| `AccountPluginSettingsTool` | `listPluginSettings`, `createPluginSetting`, `getPluginSettingDetails`, `updatePluginSettingFields`, `getPluginSettingData` (native plugins), `updatePluginSettingData` (webhook plugins), `getPluginSettingLogs`, `getMergeVariables` (private plugins), `getPluginSettingFiles` / `importPluginSettingFiles` (private plugins), `uploadPluginSettingImage` (webhook_image plugins, base64), `deletePluginSetting` |
| `AccountMarkupTool` | `readMarkup`, `writeMarkup`, `startPreview` / `getPreview`, `startRefresh` / `getRefresh` (start answers a `job_id`, poll with get) |
| `AccountPlaylistsTool` | `listDevicePlaylist`, `addDevicePlaylistItem`, `reorderDevicePlaylist`, `copyDevicePlaylist`, `listPlaylistItems`, `updatePlaylistItem`, `deletePlaylistItem`, `getPlaylistItemSchedule`, `replacePlaylistItemSchedule`, `duplicatePlaylistItem`, `createDeviceMashup`, `getMashup`, `updateMashup` |
| `AccountRecipesTool` | `searchRecipes`, `getRecipe`, `getRecipeMarkup`, `installRecipe` |
| `APIEndpointsSearchTool` | search a catalog of free third-party APIs a private plugin can poll — not TRMNL's own API |

Things that trip agents up:

- `writeMarkup` lints the Liquid and warns on variables the plugin's data does not have — fetch data first (`startRefresh`) or push it (`updatePluginSettingData`), then write.
- A schedule window is `{"week_days": [1, 2], "start_time": "09:00", "end_time": "17:00"}` with `0` for Sunday. Keys the API does not know are dropped silently.
- `updateDevice` refuses a `refresh_interval` outside 300–86400 with a 422 that names the range.
- Another user's ids answer 404, never 403 — there is no way to tell "not yours" from "does not exist".
- A 403 names what the connection lacks: a capability (`This connection lacks the devices capability`) or a device or plugin setting it was not granted. Do not retry; tell the user and ask them to widen it — connect again for a capability, Account → Connected agents for a device or plugin setting. A limited connection's `listDevices` and `listPluginSettings` show only what it was granted.

## What's in-app-only (translate or ignore)

`agent_prompt.md` was written for TRMNL's *in-app* AI assistant (the web chat at trmnl.com). A handful of sections are about that web UI specifically and don't apply to external agents (Claude Code, Cursor, Codex, Gemini, Copilot). Translate as you read:

| Reference | What it says | What to do externally |
|---|---|---|
| `agent_prompt.md:35` | "NEVER write raw HTML/markup in your chat response" | Applies only to TRMNL's in-app chat (where markup interferes with the live editor). External agents — show markup in chat freely; users often want to review it before committing. |
| `agent_prompt.md:71, 212` | "use ask_user" | `ask_user` is an in-app tool. Externally: just ask the user a normal question. |
| `agent_prompt.md:79` | "the Self-Correction Workflow (loaded separately in this system prompt)" | The mandate to screenshot after every write still applies. Iterate; react to overflow signals in the response; look for washed images (missing `image-dither`) and empty boxes (emoji used). |
| `agent_prompt.md:101` | `write_markup` "broadcasts live update to browser editor" | Web UI artifact. `MarkupsWriteTool` writes to the database; there's no editor to update. Ignore the broadcast wording. |
| `agent_prompt.md:105-108` | Tools listed: `preview_markup`, `validate_liquid`, `version_history`, `ask_user` | None of these exist as external MCP tools. Skip workflow steps that depend on them. |

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

---

## Auto-loaded references

The body above links to these reference files for readability. The imports
below tell Gemini to inline their content into context at session start, so
the agent has the full TRMNL design system available without on-demand reads.

@./refs/agent_prompt.md
@./refs/framework_v3_guide.md
@./refs/template_guide.md
