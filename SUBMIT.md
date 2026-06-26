# Marketplace submission

Repo: **https://github.com/piyiotisk/htmx-skills**

Manifests validate with:

```bash
claude plugin validate . --strict
```

## Cursor Marketplace

1. Sign in at [cursor.com/marketplace/publish](https://cursor.com/marketplace/publish)
2. Submit repository URL: `https://github.com/piyiotisk/htmx-skills`
3. Plugin name: `htmx`
4. Category: Development / Frontend
5. License: MIT

**Until listed**, users can install from GitHub:

```bash
git clone https://github.com/piyiotisk/htmx-skills.git ~/.cursor/plugins/local/htmx-skills
```

## Claude Code (plugin directory)

1. Sign in at [platform.claude.com/plugins/submit](https://platform.claude.com/plugins/submit)
2. Submit GitHub URL: `https://github.com/piyiotisk/htmx-skills`
3. Plugin type: Skills
4. License: MIT

**Available immediately** via custom marketplace:

```bash
claude plugin marketplace add piyiotisk/htmx-skills
claude plugin install htmx@htmx-skills
```

For the **community catalog** (`claude-community`), approval is separate after directory submission. Once listed:

```bash
/plugin marketplace add anthropics/claude-plugins-community
/plugin install htmx@claude-community
```

## Codex

No public marketplace submission form yet. This repo ships:

- `.codex-plugin/plugin.json` — plugin manifest
- `.agents/plugins/marketplace.json` — repo marketplace for Codex

Users install with:

```bash
git clone https://github.com/piyiotisk/htmx-skills.git
cp -R htmx-skills/skills/htmx ~/.agents/skills/htmx
```

Or point Codex at the repo marketplace (when supported in their Codex version).

Optional: open a PR to [hashgraph-online/awesome-ai-plugins](https://github.com/hashgraph-online/awesome-ai-plugins) for community discovery (requires plugin-scanner score ≥ 80).

## Submission copy-paste

**Name:** htmx  
**Description:** Build interactive web UIs with htmx — hypermedia attributes, HTML fragment swaps, OOB updates, extensions, and server-driven UI patterns. Bundles the full official htmx reference docs.  
**Repository:** https://github.com/piyiotisk/htmx-skills  
**License:** MIT  
**Keywords:** htmx, hypermedia, html, frontend, hx-get, hx-post, hx-swap

**Example usage:**

```text
/htmx:htmx Add debounced live search — hx-get /search, swap into #results.

/htmx:htmx Delete table row with hx-delete (closest tr) and OOB-update #count.

/htmx:htmx Review this template for boost inheritance and XSS issues.
```

See `skills/htmx/examples.md` for full HTML + server response examples.
