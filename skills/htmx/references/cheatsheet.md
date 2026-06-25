# htmx Reference

Detailed tables for attributes, headers, events, the JS API, response handling, and configuration. See `SKILL.md` for the conceptual guide.

## Attribute Reference

**Core attributes** (most common):

| Attribute | Description |
| --- | --- |
| `hx-get` / `hx-post` | issue a `GET` / `POST` to the URL |
| `hx-on*` | handle events with inline scripts on elements |
| `hx-push-url` | push a URL into the location bar to create history |
| `hx-select` | select content to swap in from a response |
| `hx-select-oob` | select response content to swap out of band |
| `hx-swap` | how content swaps in (`outerHTML`, `beforeend`, …) |
| `hx-swap-oob` | mark response element to swap out of band |
| `hx-target` | target element to be swapped |
| `hx-trigger` | event that triggers the request |
| `hx-vals` | add values to submit with the request (JSON) |

**Additional attributes:**

| Attribute | Description |
| --- | --- |
| `hx-boost` | progressive enhancement for links and forms |
| `hx-confirm` | `confirm()` dialog before a request |
| `hx-delete` / `hx-patch` / `hx-put` | issue that verb to the URL |
| `hx-disable` | disable htmx processing for a node and children |
| `hx-disabled-elt` | add `disabled` to elements while a request is in flight |
| `hx-disinherit` | disable automatic attribute inheritance for children |
| `hx-encoding` | change the request encoding type |
| `hx-ext` | extensions to use for this element |
| `hx-headers` | add headers submitted with the request |
| `hx-history` | prevent sensitive data being saved to history cache |
| `hx-history-elt` | element to snapshot/restore during history navigation |
| `hx-include` | include additional data in requests |
| `hx-indicator` | element to put the `htmx-request` class on |
| `hx-inherit` | re-enable inheritance when disabled by default |
| `hx-params` | filter parameters submitted with a request |
| `hx-preserve` | keep elements unchanged between requests |
| `hx-prompt` | `prompt()` before submitting a request |
| `hx-replace-url` | replace the URL in the location bar |
| `hx-request` | configure aspects of the request |
| `hx-sync` | synchronize requests across elements |
| `hx-validate` | force elements to validate before a request |
| `hx-vars` | dynamic param values (deprecated — use `hx-vals`) |

## CSS Classes

| Class | Description |
| --- | --- |
| `htmx-added` | on new content before swap, removed after settle |
| `htmx-indicator` | toggles visible (opacity:1) when `htmx-request` is present |
| `htmx-request` | on the element (or `hx-indicator` target) while a request is ongoing |
| `htmx-settling` | on a target after swap, removed after settle (duration via `hx-swap`) |
| `htmx-swapping` | on a target before swap, removed after swap (duration via `hx-swap`) |

## Request Headers (htmx → server)

| Header | Description |
| --- | --- |
| `HX-Boosted` | request is via an element using `hx-boost` |
| `HX-Current-URL` | current URL of the browser |
| `HX-History-Restore-Request` | `true` if request is for history restoration after a local cache miss |
| `HX-Prompt` | user response to an `hx-prompt` |
| `HX-Request` | always `true` (except history restore if `historyRestoreAsHxRequest` disabled) |
| `HX-Target` | `id` of the target element, if any |
| `HX-Trigger-Name` | `name` of the triggered element, if any |
| `HX-Trigger` | `id` of the triggered element, if any |

Use `HX-Request` (with `Vary: HX-Request`) to decide between returning a full page vs. a fragment.

## Response Headers (server → htmx)

| Header | Effect |
| --- | --- |
| `HX-Location` | client-side redirect without a full page reload |
| `HX-Push-Url` | push a new URL onto the history stack |
| `HX-Redirect` | client-side redirect to a new location |
| `HX-Refresh` | `true` → full client-side page refresh |
| `HX-Replace-Url` | replace the current URL in the location bar |
| `HX-Reswap` | override the swap strategy (see `hx-swap` values) |
| `HX-Retarget` | CSS selector to change the swap target |
| `HX-Reselect` | CSS selector to choose response content to swap (overrides `hx-select`) |
| `HX-Trigger` | trigger client-side events |
| `HX-Trigger-After-Settle` | trigger client-side events after the settle step |
| `HX-Trigger-After-Swap` | trigger client-side events after the swap step |

Note: these response headers are **not** processed on 3xx redirects (the browser follows the redirect). Prefer a `200` with `HX-Location`/`HX-Redirect`.

CORS: expose htmx headers via `Access-Control-Allow-Headers` (requests) and `Access-Control-Expose-Headers` (responses).

## Response Handling

htmx iterates `htmx.config.responseHandling` in order; the first entry whose `code` regex matches the status decides the behavior.

Entry fields: `code` (regex string), `swap` (bool), `error` (bool), `ignoreTitle` (bool), `select` (CSS), `target` (CSS), `swapOverride` (swap string).

Default:

```js
responseHandling: [
  { code: "204", swap: false },                 // 204: no-op, not an error
  { code: "[23]..", swap: true },               // 2xx & 3xx: swap
  { code: "[45]..", swap: false, error: true }, // 4xx & 5xx: error, no swap
  { code: "...", swap: false }                  // catch-all
]
```

Example: allow `422` validation responses to swap, configured via meta tag:

```html
<meta name="htmx-config" content='{
  "responseHandling":[
    {"code":"204","swap":false},
    {"code":"[23]..","swap":true},
    {"code":"422","swap":true},
    {"code":"[45]..","swap":false,"error":true},
    {"code":"...","swap":true}
  ]
}' />
```

## Request Lifecycle Classes

During a request/swap htmx toggles classes you can target with CSS transitions:

- `htmx-request` — on the requesting element (or `hx-indicator` target) while in flight
- `htmx-swapping` — on the target during the swap
- `htmx-added` — on each new content node
- `htmx-settling` — on the target during the settle delay (default 20ms)

## Key Events

- `htmx:load` / `htmx.onLoad(fn)` — content loaded into the DOM (init 3rd-party libs)
- `htmx:configRequest` — modify `detail.parameters` / `detail.headers` before send
- `htmx:beforeSwap` — set `detail.shouldSwap`, `detail.isError`, `detail.target`
- `htmx:beforeHistorySave` — clean up DOM mutations before a history snapshot
- `htmx:responseError` / `htmx:sendError` — server error / connection error
- `htmx:validation:validate` / `:failed` / `:halted` — validation hooks
- `htmx:validateUrl` — inspect `detail.url` / `detail.sameHost`, `preventDefault()` to block
- `htmx:confirm` — async confirmation; `evt.detail.issueRequest()` to proceed
- `htmx:abort` — send to an element to cancel in-flight requests

Register with `document.body.addEventListener('htmx:...', fn)` or `htmx.on('htmx:...', fn)`. Every event is dispatched in both camelCase and kebab-case.

### Full Event Reference

| Event | When |
| --- | --- |
| `htmx:abort` | send to an element to abort a request |
| `htmx:afterOnLoad` | after an AJAX request finishes processing a successful response |
| `htmx:afterProcessNode` | after htmx initializes a node |
| `htmx:afterRequest` | after an AJAX request completes |
| `htmx:afterSettle` | after the DOM has settled |
| `htmx:afterSwap` | after new content has been swapped in |
| `htmx:beforeCleanupElement` | before htmx disables/removes an element |
| `htmx:beforeOnLoad` | before any response processing |
| `htmx:beforeProcessNode` | before htmx initializes a node |
| `htmx:beforeRequest` | before an AJAX request is made |
| `htmx:beforeSwap` | before a swap; lets you configure the swap |
| `htmx:beforeSend` | just before an ajax request is sent |
| `htmx:beforeTransition` | before a View Transition-wrapped swap |
| `htmx:configRequest` | before the request; customize params/headers |
| `htmx:confirm` | after a trigger; cancel/delay the request |
| `htmx:historyCacheError` | error during cache writing |
| `htmx:historyCacheHit` | history cache hit |
| `htmx:historyCacheMiss` | history cache miss |
| `htmx:historyCacheMissLoadError` | unsuccessful remote retrieval |
| `htmx:historyCacheMissLoad` | successful remote retrieval |
| `htmx:historyRestore` | htmx handles a history restoration |
| `htmx:beforeHistorySave` | before content is saved to history cache |
| `htmx:load` | new content added to the DOM |
| `htmx:noSSESourceError` | SSE trigger with no parent SSE source |
| `htmx:onLoadError` | exception during onLoad handling |
| `htmx:oobAfterSwap` | after an OOB element is swapped in |
| `htmx:oobBeforeSwap` | before an OOB element swap; configurable |
| `htmx:oobErrorNoTarget` | OOB element has no matching ID in the DOM |
| `htmx:prompt` | after a prompt is shown |
| `htmx:pushedIntoHistory` | after a URL is pushed into history |
| `htmx:replacedInHistory` | after a URL is replaced in history |
| `htmx:responseError` | HTTP response error (non-2xx/3xx) |
| `htmx:sendAbort` | a request is aborted |
| `htmx:sendError` | network error prevents the request |
| `htmx:sseError` | error with an SSE source |
| `htmx:sseOpen` | SSE source opened |
| `htmx:swapError` | error during the swap phase |
| `htmx:targetError` | invalid target specified |
| `htmx:timeout` | request timeout |
| `htmx:validation:validate` | before an element is validated |
| `htmx:validation:failed` | element fails validation |
| `htmx:validation:halted` | request halted due to validation errors |
| `htmx:xhr:abort` | ajax request aborts |
| `htmx:xhr:loadend` | ajax request ends |
| `htmx:xhr:loadstart` | ajax request starts |
| `htmx:xhr:progress` | periodic during a progress-capable ajax request |

## JavaScript API

| Method / Property | Description |
| --- | --- |
| `htmx.addClass(elt, class)` | add a class to an element |
| `htmx.ajax(verb, url, target)` | issue an htmx-style ajax request |
| `htmx.closest(elt, selector)` | closest parent matching the selector |
| `htmx.config` | the current htmx config object |
| `htmx.createEventSource` | function used to create SSE EventSource objects |
| `htmx.createWebSocket` | function used to create WebSocket objects |
| `htmx.defineExtension(name, ext)` | define an htmx extension |
| `htmx.find(selector)` | first element matching the selector |
| `htmx.findAll([elt,] selector)` | all elements matching the selector |
| `htmx.logAll()` | log all htmx events |
| `htmx.logger` | current logger (default `null`) |
| `htmx.off(...)` | remove an event listener |
| `htmx.on(...)` | add an event listener (returns it) |
| `htmx.onLoad(fn)` | add a handler for the `htmx:load` event |
| `htmx.parseInterval(str)` | parse an interval declaration to ms |
| `htmx.process(elt)` | hook up htmx behavior on an element + children |
| `htmx.remove(elt)` | remove an element |
| `htmx.removeClass(elt, class)` | remove a class |
| `htmx.removeExtension(name)` | remove an extension |
| `htmx.swap(target, content, swapSpec)` | swap (and settle) HTML content |
| `htmx.takeClass(elt, class)` | take a class from sibling elements |
| `htmx.toggleClass(elt, class)` | toggle a class |
| `htmx.trigger(elt, event[, detail])` | trigger an event on an element |
| `htmx.values(elt)` | input values associated with an element |

## Configuration (`htmx.config`)

Set in JS or via `<meta name="htmx-config" content='{"defaultSwapStyle":"outerHTML"}'>`.

| Variable | Default / Info |
| --- | --- |
| `historyEnabled` | `true` (mostly for testing) |
| `historyCacheSize` | `10` |
| `refreshOnHistoryMiss` | `false`; `true` → full page refresh on history miss |
| `defaultSwapStyle` | `innerHTML` |
| `defaultSwapDelay` | `0` |
| `defaultSettleDelay` | `20` (ms) |
| `includeIndicatorStyles` | `true` (load indicator CSS) |
| `indicatorClass` | `htmx-indicator` |
| `requestClass` | `htmx-request` |
| `addedClass` | `htmx-added` |
| `settlingClass` | `htmx-settling` |
| `swappingClass` | `htmx-swapping` |
| `allowEval` | `true`; `false` disables eval features (filters, `hx-on:`, `js:` vals/headers) |
| `allowScriptTags` | `true` (process `<script>` in new content) |
| `inlineScriptNonce` | `''` |
| `inlineStyleNonce` | `''` |
| `attributesToSettle` | `["class","style","width","height"]` |
| `useTemplateFragments` | `false` (not IE11 compatible) |
| `wsReconnectDelay` | `full-jitter` |
| `wsBinaryType` | `blob` |
| `disableSelector` | `[hx-disable], [data-hx-disable]` |
| `withCredentials` | `false` (send cookies/auth on cross-site) |
| `timeout` | `0` (ms; 0 = no timeout) |
| `scrollBehavior` | `instant` (`instant`/`smooth`/`auto`) |
| `defaultFocusScroll` | `false` |
| `getCacheBusterParam` | `false`; `true` appends cache-buster to GET requests |
| `globalViewTransitions` | `false`; `true` uses the View Transition API for swaps |
| `methodsThatUseUrlParams` | `["get","delete"]` |
| `selfRequestsOnly` | `true` (only same-domain AJAX) |
| `ignoreTitle` | `false` |
| `disableInheritance` | disables attribute inheritance (opt back in via `hx-inherit`) |
| `scrollIntoViewOnBoost` | `true` |
| `triggerSpecsCache` | `null` |
| `responseHandling` | per-status swap/error config (see above) |
| `allowNestedOobSwaps` | `true` |
| `historyRestoreAsHxRequest` | `true`; disable when using `HX-Request` to return partials |
| `reportValidityOfForms` | `false`; set `true` to report validation errors and focus the first invalid input (matches native browser behavior) |

## Caching

Works with standard HTTP caching. If the same URL returns different content based on `HX-Request`, add `Vary: HX-Request` (or set `getCacheBusterParam: true`). Generate distinct `ETag`s per content variant. Disable `historyRestoreAsHxRequest` so full-page history requests aren't cached as partials.
