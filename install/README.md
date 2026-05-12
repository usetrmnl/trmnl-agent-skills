# Installing trmnl-agent-skills

One skill (`trmnl`), five harness install paths. `dist/` is committed — every harness ships with bundled reference files, no Ruby needed.

Most installs are `git clone` then `cp -r` to copy both the main file AND its sibling `refs/` directory. The `refs/` directory contains the verbatim copies of `agent_prompt.md`, `template_guide.md`, and `framework_v3_guide.md` (~3300 lines total). Without it, your agent can't read the design system on demand.

## Claude Code

```
/plugin marketplace add usetrmnl/trmnl-agent-skills
/plugin install trmnl@trmnl-agent-skills
```

This reads [`.claude-plugin/marketplace.json`](../.claude-plugin/marketplace.json) and installs the bundled plugin manifest at `dist/claude-code/.claude-plugin/plugin.json`. The skill plus all references are copied into your Claude config.

## Cursor (2.5+)

The plugin ships skills only (`SKILL.md` + references) — MCP is a separate one-time setup.

```bash
git clone https://github.com/usetrmnl/trmnl-agent-skills ~/Documents/GitHub/trmnl-agent-skills
mkdir -p ~/.cursor/plugins/local
ln -sfn ~/Documents/GitHub/trmnl-agent-skills/dist/cursor ~/.cursor/plugins/local/trmnl
```

Restart Cursor. The plugin shows up under Settings → Plugins. Edits to your clone are picked up on next Cursor restart — useful if you're contributing back.

### Wire up the TRMNL MCP server (optional, for live plugin operations)

The skill works offline as a design reference without MCP. To unlock live operations (read/write markup, screenshots, merge variable inspection), edit `~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (per-project) and add:

```json
{
  "mcpServers": {
    "trmnl": {
      "url": "https://trmnl.com/mcp?api_key=<your-key>",
      "type": "http"
    }
  }
}
```

Get your key from TRMNL dashboard → any plugin → settings → MCP. Merge this alongside any other MCP servers you already have. Restart Cursor. Check Settings → Tools & MCP for a green `trmnl` indicator.

## Codex

```bash
git clone --depth 1 https://github.com/usetrmnl/trmnl-agent-skills /tmp/trmnl
cp /tmp/trmnl/dist/codex/AGENTS.md .
cp -r /tmp/trmnl/dist/codex/refs .
```

`AGENTS.md` references `refs/<name>.md` so both must sit at the same level (project root, or wherever Codex's `AGENTS.md` lives).

If you already have an `AGENTS.md`, append + add refs separately:

```bash
cat /tmp/trmnl/dist/codex/AGENTS.md >> AGENTS.md
cp -r /tmp/trmnl/dist/codex/refs .
```

## Gemini

```bash
git clone --depth 1 https://github.com/usetrmnl/trmnl-agent-skills /tmp/trmnl
cp /tmp/trmnl/dist/gemini/GEMINI.md .
cp -r /tmp/trmnl/dist/gemini/refs .
```

Same shape as Codex. `GEMINI.md` references `refs/<name>.md` relatively.

If you set `context.fileName: ["AGENTS.md", "GEMINI.md"]` in `.gemini/settings.json`, you can reuse the Codex install instead.

## Copilot

```bash
git clone --depth 1 https://github.com/usetrmnl/trmnl-agent-skills /tmp/trmnl
cp -r /tmp/trmnl/dist/copilot/.github/* .github/
```

This copies `.github/copilot-instructions.md`, `.github/instructions/trmnl.instructions.md`, and `.github/refs/*.md`. The per-skill instruction file references `../refs/<name>.md` relatively, so the layout must be preserved.

## Verifying

After install, ask your agent: "what trmnl skill is loaded?" It should mention the `trmnl` skill and reference `template_guide.md` / `agent_prompt.md` / `framework_v3_guide.md`.

If you want to test that refs are accessible: ask the agent to "show me the first 20 lines of the trmnl template guide" — it should be able to read `refs/template_guide.md` (or the appropriate harness path) directly.

## Updating

TRMNL updates this skill whenever its production prompts change (new framework classes, new MCP tools, design rule tweaks). Refresh on your end:

### Claude Code

```
/plugin marketplace update trmnl-agent-skills
/plugin update trmnl
```

Then restart Claude Code so the new skill files load. `marketplace update` re-pulls the manifest; `plugin update` re-installs the latest version listed there.

Need to start over from scratch? `/plugin uninstall trmnl` then `/plugin marketplace remove trmnl-agent-skills`, then re-run the install commands at the top of this file.

### Cursor (2.5+)

Cursor manages plugin updates and removal through the **Settings → Plugins** panel. Open it, find `trmnl`, and click update or remove. To reinstall after removal, re-run the install steps at the top of this file. Restart Cursor afterward.

### Codex, Gemini, Copilot

Re-run the same install commands — they overwrite the previous files in place:

```bash
git clone --depth 1 https://github.com/usetrmnl/trmnl-agent-skills /tmp/trmnl
# then re-run the cp -r line for your harness
```

The git clone always pulls the latest `main` branch, so the copy is fresh every time.

## How references load per harness

The single `SKILL.md`/main-file body is what each harness loads on trigger. The 3 reference files (`agent_prompt.md`, `template_guide.md`, `framework_v3_guide.md`) sit alongside as bundled siblings:

| Harness | Main file location | Refs location | How agents access refs |
|---|---|---|---|
| Claude Code | `~/.claude/skills/trmnl/SKILL.md` | `~/.claude/skills/trmnl/references/` | Native progressive disclosure (Anthropic Skills standard) |
| Cursor (2.5+) | `<plugin-root>/skills/trmnl/SKILL.md` | `<plugin-root>/skills/trmnl/references/` | Native plugin skill discovery (Cursor 2.5+ Plugin spec) |
| Codex | `AGENTS.md` (project root) | `refs/` (project root) | Agent reads via filesystem |
| Gemini | `GEMINI.md` (project root) | `refs/` (project root) | Agent reads via filesystem (or use `@./refs/<name>.md` for explicit imports) |
| Copilot | `.github/instructions/trmnl.instructions.md` | `.github/refs/` | Agent reads via filesystem |

All non-Claude harnesses ship the same 3 reference files alongside their main file — full offline operation, no GitHub URL fetching required.
