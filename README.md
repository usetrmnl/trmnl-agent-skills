# trmnl-agent-skills

A template writing agent skill for the [TRMNL](https://trmnl.com), packaged for Claude Code, Cursor, opencode, OpenAI Codex CLI, Gemini CLI, GitHub Copilot, and Hermes Agent. Pair it with TRMNL's hosted MCP server for live operations on one plugin (API key) or your whole account (OAuth) — setup is in [`skills/trmnl/SKILL.md`](skills/trmnl/SKILL.md#adding-the-trmnl-mcp-server-optional), and the [MCP Server help article](https://help.trmnl.com/en/articles/17432548-mcp-server) walks through it for each client.

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
| **Cursor** | One click: [Add TRMNL to Cursor](https://cursor.com/install-mcp?name=trmnl&config=eyJ1cmwiOiJodHRwczovL3RybW5sLmNvbS9tY3AifQ%3D%3D) (MCP server). For the plugin with the skill, install via symlink for local dev — see [`install/README.md`](install/README.md#cursor-25). |
| **VS Code** | One click: [Add TRMNL to VS Code](https://vscode.dev/redirect/mcp/install?name=trmnl&config=%7B%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Ftrmnl.com%2Fmcp%22%7D) (MCP server) |
| **opencode** | Already installed for Claude Code? opencode reads the same Agent Skills format from `~/.claude/skills/` — nothing to do.<br>Otherwise symlink `dist/claude-code/skills/trmnl` into `~/.config/opencode/skills/` — see [`install/README.md`](install/README.md#opencode). |
| **OpenAI Codex** | drop `dist/codex/AGENTS.md` into your project root |
| **Gemini CLI** | `gemini extensions install https://github.com/usetrmnl/trmnl-agent-skills` (skill + MCP server), or drop `dist/gemini/GEMINI.md` into your project root |
| **GitHub Copilot** | copy `dist/copilot/.github/` into your repo |
| **Hermes Agent** | copy or symlink `skills/trmnl` into `$HERMES_HOME/skills/trmnl` (defaults to `~/.hermes/skills/trmnl`) |

Full install: [`install/README.md`](install/README.md).

## Use it across a team (Claude Code)

To give everyone working in a repository the TRMNL plugin, commit this to the repository's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "trmnl-agent-skills": {
      "source": { "source": "github", "repo": "usetrmnl/trmnl-agent-skills" }
    }
  },
  "enabledPlugins": {
    "trmnl@trmnl-agent-skills": true
  }
}
```

Each teammate then trusts the folder and runs this once:

```bash
claude plugin install trmnl@trmnl-agent-skills --scope project
```

New versions arrive when the plugin's version changes, which `bin/sync-from-core` bumps whenever the reference files change.

## Where TRMNL is listed

The TRMNL MCP server (`https://trmnl.com/mcp`, OAuth) is listed in:

- [Claude directory](https://claude.ai/directory/trmnl)
- [Official MCP Registry](https://registry.modelcontextprotocol.io/v0.1/servers/com.trmnl%2Ftrmnl/versions) as `com.trmnl/trmnl`
- [Smithery](https://smithery.ai/servers/trmnl/trmnl)
- [Glama](https://glama.ai/mcp/connectors/com.trmnl/trmnl)
- [mcp.so](https://mcp.so/servers/trmnl)

Setup for every client: [MCP Server help article](https://help.trmnl.com/en/articles/17432548-mcp-server).

## Develop

```bash
bin/sync-from-core    # pull latest reference files from TRMNL core (sibling repo)
bin/generate          # regenerate dist/ from skills/
```

Run `bin/generate` after editing anything under `skills/` or after `bin/sync-from-core`. Commit `dist/` alongside source so users without Ruby can install directly.

## License

MIT. See [`LICENSE`](LICENSE).
