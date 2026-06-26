---
name: htmx
description: Build interactive web UIs with htmx by adding hypermedia attributes to HTML and returning HTML fragments from the server. Use when writing or reviewing htmx code, working with hx-* attributes (hx-get, hx-post, hx-trigger, hx-target, hx-swap, hx-boost), wiring AJAX/SSE/WebSocket-driven HTML, or designing server endpoints that respond with HTML partials.
---

# htmx

htmx lets any element issue an AJAX request on any event using any HTTP verb, and swap the returned **HTML** into any target. The server returns HTML fragments, not JSON. Keep logic in HTML attributes (Locality of Behavior) and keep state on the server (HATEOAS).

## Core Mental Model

1. An element is **triggered** by an event (`hx-trigger`).
2. It issues an HTTP request via `hx-get` / `hx-post` / `hx-put` / `hx-patch` / `hx-delete`.
3. The server responds with an **HTML fragment**.
4. htmx swaps that HTML into a **target** (`hx-target`) using a **swap strategy** (`hx-swap`).

```html
<button hx-post="/clicked" hx-trigger="click" hx-target="#out" hx-swap="outerHTML">
  Click Me!
</button>
<div id="out"></div>
```

You can use the `data-hx-*` prefix if you need valid-by-DTD HTML (e.g. `data-hx-post`).

## Example usage

Invoke explicitly when starting htmx work (Claude Code: `/htmx:htmx`).

| Task | Example prompt |
|------|----------------|
| Live search | "Add debounced `hx-get` search swapping into `#results`." |
| Inline edit | "Click Edit replaces this card with a form; Cancel restores it." |
| Delete + counter | "Delete row with `hx-delete` and OOB-update `#contact-count`." |
| Validation UX | "422 errors in `#form-errors`, success replaces the form." |
| Boosted nav | "Boost `#main` links but keep sidebar PDFs as full page loads." |
| Code review | "Review this template for htmx inheritance and XSS issues." |

For full HTML + server response examples, see [examples.md](examples.md). For pattern recipes, see `references/examples/`.

## Installing

CDN (fastest start; pin the version, consider self-hosting in production):

```html
<script src="https://cdn.jsdelivr.net/npm/htmx.org@2.0.10/dist/htmx.min.js"
  integrity="sha384-H5SrcfygHmAuTDZphMHqBJLc3FhssKjG7w/CeCpFReSfwBWDTKpkzPP8c+cLsK+V"
  crossorigin="anonymous"></script>
```

npm: `npm install htmx.org@2.0.10`. With bundlers, also expose it globally if needed: `window.htmx = require('htmx.org')`.

## AJAX Attributes

| Attribute | Request |
| --- | --- |
| `hx-get` / `hx-post` / `hx-put` / `hx-patch` / `hx-delete` | issues that verb to the URL |

Default trigger events: `input`/`textarea`/`select` → `change`; `form` → `submit`; everything else → `click`.

## Triggers (`hx-trigger`)

Override the natural event and add modifiers (comma-separate multiple triggers):

```html
<input name="q" hx-get="/search" hx-trigger="keyup changed delay:500ms"
       hx-target="#results">
<div id="results"></div>
```

Common modifiers: `once`, `changed`, `delay:<time>` (debounce, resets on new event), `throttle:<time>` (fires at end, discards intermediate), `from:<selector>` (listen on another element).

- **Filters:** `hx-trigger="click[ctrlKey]"` — JS expression in `[]`; must be truthy. Props resolve against the event first, then global scope; `this` is the element.
- **Special events:** `intersect` (with `root:`/`threshold:`), `load`, `revealed`.
- **Polling:** `hx-trigger="every 2s"`. Server can respond `286` to stop polling. "Load polling": `hx-trigger="load delay:1s" hx-swap="outerHTML"` to re-fetch a self-replacing element.

## Targets (`hx-target`)

Takes a CSS selector, plus extended syntax: `this`, `closest <sel>`, `next <sel>`, `previous <sel>`, `find <sel>`. Prefer relative targets over peppering the DOM with IDs.

## Swapping (`hx-swap`)

Default is `innerHTML`. Options: `innerHTML`, `outerHTML`, `afterbegin`, `beforebegin`, `beforeend`, `afterend`, `delete`, `none`.

Colon-separated modifiers after the strategy: `transition:true`, `swap:<time>`, `settle:<time>`, `ignoreTitle:true`, `scroll:top|bottom`, `show:top|bottom`. Example: `hx-swap="outerHTML show:top"`.

- **Out-of-band swaps:** put `hx-swap-oob="true"` on an element in the *response* to swap it by `id` outside the main target (wrap `tr`/`td` etc. in `<template>`). Piggy-back extra updates on a response.
- **Select subset:** `hx-select` (CSS selector of response content to use); `hx-select-oob` (IDs to OOB-swap).
- **Preserve:** `hx-preserve` keeps an element (e.g. a playing video) across swaps.
- **Morphing** (preserves focus/state) is available via the `idiomorph` extension (`hx-swap="morph"`).

## Forms, Parameters & Values

- Requesting elements include their own value; non-GET requests include the enclosing form's inputs (by `name`).
- `hx-include="<sel>"` — include other elements' values. `hx-params` — filter params.
- `hx-vals='{"k":"v"}'` — extra static values (use `js:` prefix for computed). 
- File upload: `hx-encoding="multipart/form-data"`; hook `htmx:xhr:progress` for progress.
- `hx-confirm="Are you sure?"` for a confirm dialog; for custom dialogs handle the `htmx:confirm` event and call `evt.detail.issueRequest()`.

## Request Indicators

Add `htmx-indicator` class to a spinner; htmx adds `htmx-request` to the requesting element during the request, revealing it. Point elsewhere with `hx-indicator="#sel"`. Disable elements during requests with `hx-disabled-elt`.

## Synchronization

`hx-sync="<sel>:<strategy>"` coordinates requests, e.g. `hx-sync="closest form:abort"` aborts an input's request when its form submits. Cancel programmatically by sending the `htmx:abort` event to an element.

## Boosting & Progressive Enhancement

`hx-boost="true"` turns child anchors/forms into AJAX requests targeting `body`, degrading gracefully without JS. Wrap htmx inputs in real `<form action method>` so non-JS clients still work. Check the `HX-Request` header server-side to decide between full pages and fragments.

## Attribute Inheritance

Most `hx-*` attributes are inherited by descendants, so hoist shared ones (e.g. `hx-confirm`) to a parent. Undo with `hx-confirm="unset"`, control with `hx-disinherit` / `hx-inherit`, or globally via `htmx.config.disableInheritance`.

## History

`hx-push-url="true"` pushes the request URL to the browser history (snapshots the DOM). The URL **must** return a full page when visited directly. Narrow the snapshot with `hx-history-elt`; disable caching sensitive pages with `hx-history="false"`.

## Requests & Responses

- Respond with HTML fragments (or a full doc + `hx-select`). `204 No Content` swaps nothing.
- Error responses (4xx/5xx) fire `htmx:responseError`; connection failures fire `htmx:sendError`.
- No need for Post/Redirect/Get — return the new HTML directly. Note: htmx response headers are **not** processed on 3xx redirects (the browser follows them); prefer `200` + `HX-Location`/`HX-Redirect`.
- To swap on a normally-ignored code (e.g. `422` validation), configure `htmx.config.responseHandling` or use the `response-targets` extension.

See `references/cheatsheet.md` for the full request/response header tables and `responseHandling` config, or `references/headers/` for per-header behavior.

## Scripting (`hx-on:*`)

Respond to *any* event inline: `hx-on:click="..."` or `hx-on:htmx:config-request="event.detail.parameters.x = 1"`. Attribute names are case-insensitive, so htmx fires every event in **both** camelCase and kebab-case (`htmx:afterSwap` / `htmx:after-swap`).

Pairs well with VanillaJS, Alpine.js, jQuery, hyperscript. For deeper logic prefer those over `hx-on`.

## Integrating 3rd-Party JS

Initialize new content with `htmx.onLoad(fn)` (scope queries to the loaded content, not `document`). If you inject htmx-attributed HTML yourself (e.g. via `fetch`), call `htmx.process(el)` so htmx wires it up. Clean up library mutations before history snapshots via `htmx:beforeHistorySave`.

## Validation

htmx respects the HTML5 Validation API and won't submit invalid forms. Hook `htmx:validation:validate` / `:failed` / `:halted`. Enable validation on non-form elements with `hx-validate="true"`. Set `htmx.config.reportValidityOfForms = true` to restore native error reporting. Always re-validate on the server.

## Extensions

Enable via `hx-ext="name"` on an ancestor. Core extensions: `head-support`, `htmx-1-compat`, `idiomorph`, `preload`, `response-targets`, `sse`, `ws`. Load the extension script *after* htmx core.

## Security (essentials)

- **Escape all user content** — htmx makes injected HTML more dangerous (XSS). Whitelist allowed tags/attributes; strip `hx-*`/`data-hx-*` from untrusted HTML.
- `hx-disable` stops htmx processing on an element and its children (cannot be re-enabled by injected content).
- `htmx.config.selfRequestsOnly` (default `true`) and `htmx:validateUrl` restrict request destinations; add a CSP for defense in depth.
- `htmx.config.allowEval = false` disables eval-based features (trigger filters, `hx-on:`, `js:` vals/headers).
- CSRF: attach a token via `hx-headers='{"X-CSRF-TOKEN":"..."}'` on `<html>`/`<body>` (note `hx-boost` doesn't replace head/body).

## Debugging

`htmx.logAll()` logs every event. In the browser console, `monitorEvents(htmx.find('#el'))` shows what events an element fires (console only). Set `htmx.logger` for a custom logger.

## Reference Library

This skill bundles the full official htmx documentation under `references/`. The guidance above covers the common cases; when you need authoritative detail, read the relevant file rather than guessing. Don't load everything — open only what the task needs.

**Quick lookup**
- `references/cheatsheet.md` — condensed tables: attributes, CSS classes, request/response headers, full event list, JS API, `responseHandling`, and every `htmx.config` option. Start here for a fast answer.

**Authoritative docs**
- `references/docs.md` — the complete official narrative guide (the `/docs/` page in full).
- `references/reference.md` — the official API reference (the `/reference/` page).
- `references/api.md` — the JavaScript API in detail (`htmx.ajax`, `htmx.process`, `htmx.on`, `htmx.swap`, …).
- `references/events.md` — every htmx event with `event.detail` payloads.

**Per-attribute deep docs** — `references/attributes/<name>.md`, one file per attribute, with modifiers, edge cases, and examples. Read these when the cheatsheet isn't enough. High-value ones:
- `hx-trigger.md` (the largest — all modifiers, filters, polling, `from:`, special events)
- `hx-swap.md` / `hx-swap-oob.md` (swap strategies, modifiers, OOB rules)
- `hx-target.md`, `hx-sync.md`, `hx-on.md`, `hx-vals.md`, `hx-headers.md`, `hx-include.md`, `hx-indicator.md`, `hx-push-url.md`, `hx-boost.md`, `hx-disinherit.md` / `hx-inherit.md`.

**Response headers** — `references/headers/<name>.md`: `hx-location.md`, `hx-redirect.md`, `hx-push-url.md`, `hx-replace-url.md`, `hx-trigger.md`. Read when designing server responses.

**Pattern recipes** — `references/examples/<name>.md`: copy-pasteable, working implementations. Reach for these when building a known UI pattern: `active-search`, `click-to-edit`, `click-to-load`, `infinite-scroll`, `inline-validation`, `delete-row`, `edit-row`, `bulk-update`, `update-other-content`, `progress-bar`, `file-upload`, `modal-custom` / `modal-bootstrap` / `modal-uikit`, `tabs-hateoas`, `lazy-load`, `sortable`, `confirm`, `dialogs`, `animations`, `keyboard-shortcuts`, `value-select`, `web-components`, `async-auth`, `reset-user-input`.

**Extensions** — `references/extensions/<name>.md`: `sse.md`, `ws.md`, `response-targets.md`, `idiomorph.md`, `preload.md`, `head-support.md`, `htmx-1-compat.md`, plus `building.md` for authoring your own.

**Gotchas & migration**
- `references/quirks.md` — surprising behaviors worth knowing before debugging.
- `references/migration-guide-htmx-1.md` — moving from htmx 1.x to 2.x.

Note: the bundled docs are the upstream Hugo markdown, so they contain occasional `{{...}}` template shortcodes — ignore those, the surrounding prose and code are accurate.
