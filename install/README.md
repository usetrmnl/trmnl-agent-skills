# Installing trmnl-agent-skills

One skill (`trmnl`), five harness install paths. `dist/` is committed — every harness ships with bundled reference files, no Ruby needed.

Most installs are `git clone` then `cp -r` to copy both the main file AND its sibling `refs/` directory. The `refs/` directory contains the verbatim copies of `agent_prompt.md`, `template_guide.md`, and `framework_v3_guide.md` (~3300 lines total). Without it, your agent can't read the design system on demand.

## Claude Code

```
/plugin marketplace add usetrmnl/trmnl-agent-skills
/plugin install trmnl@trmnl-agent-skills
```

This reads [`.claude-plugin/marketplace.json`](../.claude-plugin/marketplace.json) and installs the bundled plugin manifest at `dist/claude-code/.claude-plugin/plugin.json`. The skill plus all references are copied into your Claude config.

## Cursor

```bash
git clone --depth 1 https://github.com/usetrmnl/trmnl-agent-skills /tmp/trmnl
cp -r /tmp/trmnl/dist/cursor/.cursor/* .cursor/
```

This copies BOTH `.cursor/rules/trmnl.mdc` AND `.cursor/refs/*.md`. The rule references `../refs/<name>.md` relatively, so the refs must sit in `.cursor/refs/` (which the cp above ensures).

The rule uses **Agent Requested mode** (description-driven). It activates when you mention TRMNL in a Cursor chat. To always-apply, edit the `.mdc` frontmatter and set `alwaysApply: true`.

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

### Cursor, Codex, Gemini, Copilot

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
| Cursor | `.cursor/rules/trmnl.mdc` | `.cursor/refs/` | Agent reads via filesystem when SKILL.md links cite them |
| Codex | `AGENTS.md` (project root) | `refs/` (project root) | Agent reads via filesystem |
| Gemini | `GEMINI.md` (project root) | `refs/` (project root) | Agent reads via filesystem (or use `@./refs/<name>.md` for explicit imports) |
| Copilot | `.github/instructions/trmnl.instructions.md` | `.github/refs/` | Agent reads via filesystem |

All non-Claude harnesses ship the same 3 reference files alongside their main file — full offline operation, no GitHub URL fetching required.
