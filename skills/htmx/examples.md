# htmx — Example Usage

Use these patterns when implementing or reviewing htmx. Invoke the skill explicitly in Claude Code:

```text
/htmx:htmx
```

Or describe the task and let Claude match the skill from context.

---

## 1. Active search (debounced `hx-get`)

**Prompt:** "Add live search to this contact list — debounce 300ms, swap results into `#results`."

**HTML:**

```html
<input type="search" name="q" placeholder="Search contacts…"
       hx-get="/contacts/search"
       hx-trigger="keyup changed delay:300ms, search"
       hx-target="#results"
       hx-indicator="#search-spinner">
<img id="search-spinner" class="htmx-indicator" src="/spinner.svg" alt="">
<div id="results">
  <!-- server renders initial results here -->
</div>
```

**Server (`GET /contacts/search?q=…`):** return an HTML fragment only — rows or cards, not a full page.

```html
<div class="contact-row">…</div>
<div class="contact-row">…</div>
```

---

## 2. Click to edit (swap `outerHTML` on `this`)

**Prompt:** "Make this contact card inline-editable — click Edit loads a form, Cancel restores the card."

**Display state:**

```html
<div id="contact-1" hx-target="this" hx-swap="outerHTML">
  <p><strong>Joe Blow</strong> — joe@example.com</p>
  <button hx-get="/contacts/1/edit">Edit</button>
</div>
```

**Server (`GET /contacts/1/edit`):** return the form fragment.

```html
<form hx-put="/contacts/1" hx-target="this" hx-swap="outerHTML">
  <input name="name" value="Joe Blow">
  <input name="email" type="email" value="joe@example.com">
  <button type="submit">Save</button>
  <button type="button" hx-get="/contacts/1">Cancel</button>
</form>
```

**Server (`PUT /contacts/1`):** return the updated display card HTML (same shape as the first block).

---

## 3. Delete row + update count (OOB swap)

**Prompt:** "Delete button removes the table row and updates `#contact-count` without a second request."

**Row button:**

```html
<tr id="contact-42">
  <td>Ada Lovelace</td>
  <td>
    <button hx-delete="/contacts/42"
            hx-target="closest tr"
            hx-swap="outerHTML"
            hx-confirm="Delete this contact?">
      Delete
    </button>
  </td>
</tr>
```

**Server (`DELETE /contacts/42`):** return empty body for the main swap **and** an out-of-band count:

```html
<span id="contact-count" hx-swap-oob="true">41 contacts</span>
```

Wrap `<tr>`/`<td>` OOB payloads in `<template>` when needed (see `references/attributes/hx-swap-oob.md`).

---

## 4. Form validation errors (`422` + `response-targets`)

**Prompt:** "On validation failure, show errors in `#form-errors` but keep the form; on success swap the whole form."

Enable the extension on an ancestor:

```html
<div hx-ext="response-targets">
  <form hx-post="/contacts"
        hx-target="#form-errors"
        hx-swap="innerHTML">
    <div id="form-errors"></div>
    <input name="email" type="email" required>
    <button type="submit">Create</button>
  </form>
</div>
```

**Server:** `422` with error fragment for `#form-errors`; `200` with success fragment targeting the form via `HX-Retarget` / extension config. See `references/extensions/response-targets.md`.

---

## 5. Boosted navigation inside a page island

**Prompt:** "Boost links in `#main` but not the sidebar PDF downloads."

```html
<body hx-boost="true" hx-target="#main" hx-select="#main" hx-swap="innerHTML show:top">
  <aside hx-boost="false">
    <a href="/reports/q1.pdf">Q1 PDF</a>  <!-- full page navigation -->
  </aside>
  <main id="main">…</main>
</body>
```

**Server:** for boosted requests, check `HX-Request: true` and return a full page (htmx extracts `#main`). For direct visits, return the same full page.

Load `head-support` when swapping `<title>` and styles on boost. See `references/extensions/head-support.md`.

---

## 6. Post/redirect without a redirect (`HX-Location`)

**Prompt:** "After creating a contact, navigate to `/contacts/42` and show the detail partial."

**Server response** (`200`, not `302`):

```http
HTTP/1.1 200 OK
HX-Location: /contacts/42

<div id="main">…detail fragment…</div>
```

htmx does not process response headers on 3xx redirects — use `200` + `HX-Location` or `HX-Redirect`. See `references/headers/hx-location.md`.

---

## 7. Review an existing template

**Prompt:** "Use the htmx skill to review this template for inheritance bugs and missing progressive enhancement."

Claude should check:

- Real `<form action method>` wrappers for non-JS fallback
- `hx-target` inheritance (`hx-disinherit` / `closest` selectors)
- Escaped user content (XSS)
- Server returns fragments, not JSON
- CSRF token via `hx-headers` on `<body>` when using `hx-boost`

---

## More recipes

Full copy-paste patterns live under `references/examples/`:

| Pattern | File |
|---------|------|
| Infinite scroll | `references/examples/infinite-scroll.md` |
| Inline validation | `references/examples/inline-validation.md` |
| File upload | `references/examples/file-upload.md` |
| SSE live updates | `references/extensions/sse.md` |
| Morph swap (keep focus) | `references/extensions/idiomorph.md` |
