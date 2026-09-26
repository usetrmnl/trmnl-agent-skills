# trmnl-agent-skills

A template writing agent skill for the [TRMNL](https://trmnl.com), packaged for Claude Code, Cursor, opencode, OpenAI Codex CLI, Gemini CLI, GitHub Copilot, and Hermes Agent. Pair it with TRMNL's hosted MCP server for live operations on one plugin (API key) or your whole account (OAuth) — setup is in [`skills/trmnl/SKILL.md`](skills/trmnl/SKILL.md#adding-the-trmnl-mcp-server-optional).

The skill bundles three files copied verbatim from TRMNL's core repo, curated for the **external** agent context:

| File | What it is |
|---|---|
| `agent_prompt.md` | TRMNL AI assistant agent rules — mandatory workflows, hard rules, common mistakes. Also what TRMNL's MCP server ships as connection-level instructions to external clients. |
| `template_guide.md` | The full TRMNL design system (every framework class, layout, chart pattern; ~2700 lines). |
| `framework_v3_guide.md` | Framework v3 supplement: chromatic palette, CSS variables, label variants. |

Source of truth: [`skills/trmnl/`](skills/trmnl/). Generated outputs (committed for zero-tool install): [`dist/`](dist/).

## Install

| Harness | Command |
|---|---|
| **Claude Code** | `/plugin marketplace add usetrmnl/trmnl-agent-skills`<br>then `/plugin install trmnl@trmnl-agent-skills` |
| **Cursor** (2.5+) | Install via symlink for local dev — see [`install/README.md`](install/README.md#cursor-25). |
| **opencode** | Already installed for Claude Code? opencode reads the same Agent Skills format from `~/.claude/skills/` — nothing to do.<br>Otherwise symlink `dist/claude-code/skills/trmnl` into `~/.config/opencode/skills/` — see [`install/README.md`](install/README.md#opencode). |
| **OpenAI Codex** | drop `dist/codex/AGENTS.md` into your project root |
| **Gemini CLI** | drop `dist/gemini/GEMINI.md` into your project root |
| **GitHub Copilot** | copy `dist/copilot/.github/` into your repo |
| **Hermes Agent** | copy or symlink `dist/hermes/skills/trmnl` into `$HERMES_HOME/skills/trmnl` (defaults to `~/.hermes/skills/trmnl`) |

Full install: [`install/README.md`](install/README.md).

## Develop

```bash
bin/sync-from-core    # pull latest reference files from TRMNL core (sibling repo)
bin/generate          # regenerate dist/ from skills/
```

Run `bin/generate` after editing anything under `skills/` or after `bin/sync-from-core`. Commit `dist/` alongside source so users without Ruby can install directly.

## License

MIT. See [`LICENSE`](LICENSE).
