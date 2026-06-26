# htmx Skills

<p align="center">
  <img src="assets/lockup-dark.png" alt="htmx skills — hypermedia UI agent skill" width="480">
</p>

An open-source [Agent Skills](https://agentskills.io) plugin that teaches AI coding agents how to build hypermedia-driven UIs with [htmx](https://htmx.org).

The skill covers `hx-*` attributes, triggers, targets, swaps, out-of-band updates, extensions, security essentials, and ships the full official htmx reference docs under `skills/htmx/references/`.

## Install

### Cursor

**Marketplace** (after listing): search for **htmx** in [Cursor Marketplace](https://cursor.com/marketplace) or Customize → Plugins.

**From GitHub now:**

```bash
git clone https://github.com/piyiotisk/htmx-skills.git ~/.cursor/plugins/local/htmx-skills
```

Or in Cursor: **Customize → Plugins → Add from GitHub** → `piyiotisk/htmx-skills`

### Claude Code

**Community marketplace** (after approval): `/plugin marketplace add anthropics/claude-plugins-community` then `/plugin install htmx@claude-community`

**From this repo now:**

```bash
claude plugin marketplace add piyiotisk/htmx-skills
claude plugin install htmx@htmx-skills
```

Local test:

```bash
git clone https://github.com/piyiotisk/htmx-skills.git
claude --plugin-dir ./htmx-skills
# Skill: /htmx:htmx
```

### Codex

**From this repo:**

```bash
git clone https://github.com/piyiotisk/htmx-skills.git
# Option A: install skill folder
cp -R htmx-skills/skills/htmx ~/.agents/skills/htmx

# Option B: add repo as a plugin marketplace (Codex reads .agents/plugins/marketplace.json)
```

In Codex, invoke with `$htmx` or ask the agent to use the htmx skill.

## Example usage

### Claude Code

After installing, invoke the skill:

```text
/htmx:htmx
```

Then ask for a specific pattern:

```text
/htmx:htmx Add debounced live search on this input — hx-get /contacts/search,
swap results into #results, show a spinner with hx-indicator.

/htmx:htmx Delete button on each table row — hx-delete, closest tr target,
and OOB-update #contact-count in the same response.

/htmx:htmx Review this template for hx-boost inheritance bugs and missing
progressive-enhancement fallbacks.
```

### Cursor / Codex

Mention the skill in chat:

- "Use the htmx skill to add inline edit on this contact card."
- "Use the htmx skill to wire delete-row + OOB count update."
- "Review this form for correct `hx-sync` and validation handling."

### What the skill helps with

- `hx-get`, `hx-post`, `hx-trigger`, `hx-target`, `hx-swap`
- Server endpoints that return HTML fragments (not JSON)
- OOB swaps, boosted navigation, SSE/WebSocket extensions
- Debugging swap, history, and attribute-inheritance issues

Full worked examples (HTML + server responses): [`skills/htmx/examples.md`](skills/htmx/examples.md)

## What's included

| Path | Purpose |
|------|---------|
| `skills/htmx/SKILL.md` | Core skill instructions |
| `skills/htmx/references/` | Bundled htmx docs (attributes, headers, examples, extensions) |
| `skills/htmx/evals/` | Eval definitions used during skill development |

## Development

This repo also contains internal eval workspace under `.cursor/skills/htmx-workspace/` (not part of the published plugin).

Validate Claude plugin manifest:

```bash
claude plugin validate .
claude plugin validate . --strict
```

## Attribution

Reference documentation under `skills/htmx/references/` is derived from the [htmx](https://htmx.org) project documentation. Skill packaging and curation are MIT-licensed (see [LICENSE](LICENSE)).

## License

MIT © Constantinos Pigiotis
