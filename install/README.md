# Installing trmnl-agent-skills

One skill (`trmnl`), six harness install paths across five generated outputs — opencode shares Claude Code's, since both read the Anthropic Agent Skills format. `dist/` is committed — every harness ships with bundled reference files, no Ruby needed.

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
git clone https://github.com/usetrmnl/trmnl-agent-skills ~/trmnl-agent-skills
mkdir -p ~/.cursor/plugins/local
ln -sfn ~/trmnl-agent-skills/dist/cursor ~/.cursor/plugins/local/trmnl
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

For the whole account (devices, playlists, every plugin setting) instead of one plugin, drop `?api_key=` and Cursor prompts an OAuth sign-in on first use. Account access is opening account by account — see the "Adding the TRMNL MCP server" section of [`SKILL.md`](../skills/trmnl/SKILL.md).

## opencode

opencode reads the Anthropic Agent Skills format natively, so there's no opencode-specific build — it installs the same `dist/claude-code/skills/trmnl` directory verbatim.

**Already installed for Claude Code?** opencode also discovers skills from `~/.claude/skills/` and `.claude/skills/`, so the plugin install is picked up automatically and there's nothing to do. Otherwise:

```bash
git clone https://github.com/usetrmnl/trmnl-agent-skills ~/trmnl-agent-skills
mkdir -p ~/.config/opencode/skills
ln -sfn ~/trmnl-agent-skills/dist/claude-code/skills/trmnl \
        ~/.config/opencode/skills/trmnl
```

Restart opencode. Edits to your clone are picked up on next start — useful if you're contributing back.

Other locations opencode scans, if you prefer one of them — project-local paths win over global:

| Scope | Paths |
|---|---|
| Project | `.opencode/skills/`, `.claude/skills/`, `.agents/skills/` |
| Global | `~/.config/opencode/skills/`, `~/.claude/skills/`, `~/.agents/skills/` |

Project paths are resolved by walking up from the working directory to the git worktree root, so a skill committed at your repo root applies to every subdirectory.

### Wire up the TRMNL MCP server (optional, for live plugin operations)

Same trade-off as Cursor — the skill works offline as a design reference without MCP. opencode configures MCP in `opencode.json` (project) or `~/.config/opencode/opencode.json` (global), under `mcp` rather than `mcpServers`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "trmnl": {
      "type": "remote",
      "url": "https://trmnl.com/mcp?api_key=<your-key>",
      "enabled": true
    }
  }
}
```

Get your key from TRMNL dashboard → any plugin → settings → MCP. Merge this alongside any other MCP servers you already have, then restart opencode.

For the whole account instead of one plugin, drop `?api_key=` and run `opencode mcp auth trmnl` for the OAuth sign-in (it registers itself, no client id needed), or use your account API key from <https://trmnl.com/account>. Account access is opening account by account — see [`SKILL.md`](../skills/trmnl/SKILL.md).

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

On **opencode** this works the same way as Claude Code: relative paths in `SKILL.md` resolve against the skill's own directory, and opencode injects that base directory plus a sample of up to ten supporting file paths when the skill fires — our three `references/` files fit well inside that sample. Contents are read on demand, not preloaded.

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

### opencode

If opencode is picking up the Claude Code plugin install from `~/.claude/skills/`, update that (see above) — there's nothing separate to do here.

If you did the symlink install, `~/.config/opencode/skills/trmnl` points at your clone rather than holding a copy of it, so updating the clone updates the skill. Pull it:

```bash
cd ~/trmnl-agent-skills
git pull
```

Restart opencode. To uninstall, delete the symlink — this leaves your clone alone:

```bash
rm ~/.config/opencode/skills/trmnl
```

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
| opencode | `~/.config/opencode/skills/trmnl/SKILL.md` | `~/.config/opencode/skills/trmnl/references/` | Native Agent Skills support — agent reads refs from the skill directory |
| Codex | `AGENTS.md` (project root) | `refs/` (project root) | Agent reads via filesystem |
| Gemini | `GEMINI.md` (project root) | `refs/` (project root) | Agent reads via filesystem (or use `@./refs/<name>.md` for explicit imports) |
| Copilot | `.github/instructions/trmnl.instructions.md` | `.github/refs/` | Agent reads via filesystem |

All non-Claude harnesses ship the same 3 reference files alongside their main file — full offline operation, no GitHub URL fetching required.
