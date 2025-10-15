# Cookiebar (PrivacyGenerator CMP)

Lightweight, standalone, and accessible cookie consent modal (CMP) that you can drop into any site. It provides multilingual content, granular consent categories, cookie persistence with versioning and anonymous user IDs, Google Tag Manager dataLayer events, and optional server-side consent updates.

This README documents the behavior found in `modal-cookiebar-v2.html` and how to use it.

## Features

- **Granular categories**: `strictly-necessary`, `analytical`, `preferences`, `marketing` with per-category required and default state controls.
- **User actions**: Accept all, Deny optional, or Save selected preferences.
- **Persistence**: Stores consent in a single cookie with `groups`, `datestamp`, `version`, and a stable `userid`.
- **Versioning**: Bumps in major version invalidate old consent and prompt again.
- **Accessibility**: Proper ARIA for dialog, focus trap, keyboard support, live region updates, and labelled switches.
- **Multilingual**: Language selector and automatic browser-language detection (optional).
- **UI**: Mobile-friendly modal, CSS isolated under the `.oscbr` scope.
- **Integrations**: Pushes dataLayer events; optional server updates via `sendBeacon`/`fetch`.

## Compatibility

- Coded in ES5 (no build step required); drop-in script works without transpilation.
- Can be run and hosted directly from Google Tag Manager (Custom HTML tag).

## Quick start

1) Include the markup and script from `modal-cookiebar-v2.html` on your page (for example in a GTM Custom HTML tag, or directly in your HTML). The file already contains:

- The modal HTML with `id="oscbr"` and the consent UI
- The CSS scoped under `.oscbr`
- The `PrivacyGenerator` constructor and initialization code

2) Configure the `options` passed to `new PrivacyGenerator(options)` inside the snippet. At minimum set:

- `host`: the cookie domain (e.g. `.example.com`), or leave empty to use the current host
- `version`: semantic version string like `2.0.0`
- `modal`: the DOM element with `id="oscbr"`

3) Deploy. On first load (or after a major version change), the modal is displayed and the focus is trapped inside it until the user acts or closes it (if dismissible).

> The initialization code exposes `window.OSCBR.instance` and guards against double init when previewing via GTM.

## Configuration reference

All configuration is provided through the `options` object (as used in `modal-cookiebar-v2.html`):

- **host**: string domain for the consent cookie; empty uses current host.
- **version**: string; changing the major version invalidates existing consent.
- **modal**: HTMLElement; the root element of the modal (`#oscbr`).
- **apiKey**: optional string; required only when `sendToServer` is true.
- **consentEndpoint**: string URL; required when `sendToServer` is true.
- **cookieName**: string (default `customConsent`).
- **cookieLifetimeDays**: number (default `365`).
- **refreshDays**: number (default `365`); extends the cookie when validated.
- **dismissible**: boolean (default `true`); if `false`, user must choose.
- **multilingual**: boolean (default `false`); enables language selector.
- **langAutoSelect**: boolean (default `false`); detects browser language.
- **autoFocus**: boolean (default `true`); focuses the modal on show.
- **logLevel**: `NONE|ERROR|WARN|INFO|DEBUG` (default `INFO`).
- **texts**: array of language objects `{ lang, header, intro, buttons, categories }`.
- **delayBeforeSave**: number ms (default `500`).
- **delayModalClose**: number ms (default `1000`).
- **delayInitialFocus**: number ms (default `100`).
- **categoryConfig**: object keyed by category (`strictly-necessary`, `analytical`, `preferences`, `marketing`), each with `{ required: boolean, defaultEnabled: boolean }`.
- **sendToServer**: boolean (default `true` in code, example sets `false`); when true, consent updates are POSTed to `consentEndpoint` with `apiKey`.

### Categories and UI

- Required categories hide their switch (always on).
- Optional categories show a switch with `defaultEnabled` as initial state.
- Category accordions expand to show descriptions; switches do not toggle accordions when interacted with.

### Texts and language

Provide texts through `options.texts` (array). When multilingual is enabled:

- If `langAutoSelect` is true, the browser language is matched by exact code or language prefix.
- Otherwise, the first language object is used.
- The inline language selector lets users switch at runtime.

You can also override texts via URL parameters (useful in GTM):

- `header`, `intro`, `accept_label`, `deny_label`, `save_label`
- Per-category overrides: `required_header`, `required_body`, `analytical_header`, `analytical_body`, `preferences_header`, `preferences_body`, `marketing_header`, `marketing_body`

## Consent cookie

Consent is stored in a single cookie (default name `customConsent`) using URL-encoded key/value pairs joined with `&`:

- `groups`: colon-separated categories, e.g. `strictly-necessary:analytical`
- `datestamp`: `Date.now()` at save time
- `version`: the CMP version string
- `userid`: a stable anonymous ID (cryptographically random when available)

Behavior:

- If a cookie exists and the major version matches, it is refreshed for `refreshDays` and the `userid` is ensured.
- If the major version differs, the old cookie is removed and the modal re-opens to ask consent again.

## User actions

- **Accept**: Consents to all categories.
- **Deny**: Consents only to required categories; optional categories are off.
- **Save**: Consents to required categories plus any optional categories currently toggled on.

Each action updates the cookie and the UI, writes a live-region status message for screen readers, and can delay closing the modal via `delayBeforeSave`/`delayModalClose`.

## Accessibility

- Dialog uses `role="dialog"`, `aria-modal="true"`, `aria-labelledby`.
- Focus trap keeps tab focus inside the modal; initial focus is configurable.
- Switches use `role="switch"` with `aria-checked` updates.
- Live region (`#oscbr-consent-status`) announces changes politely.
- Keyboard interaction is respected; accordions track `aria-expanded`.

## Integrations

### Google Tag Manager / dataLayer

The CMP pushes events to `window.dataLayer`:

- `cookiebarConsentInit` on initialization
- `cookiebarConsentUpdate` after consent changes

Both include `consentedCategories` as a comma-separated string. Use these events to drive GTM triggers and variables.

### Optional server updates

Easily connect a server-side backend to register user consent decisions. Enable `sendToServer` and provide `consentEndpoint` and `apiKey`; the CMP will POST a compact payload on each change that you can persist (e.g., keyed by `userId` and `version`).

If `sendToServer` is true and both `consentEndpoint` and `apiKey` are provided, the CMP sends a POST (via `navigator.sendBeacon` when available, otherwise `fetch`) with payload:

- `userId`, `timestamp`, `change: { added, removed }`, `allConsented`, `version`, `api_key`

Requests use `Content-Type: text/plain;charset=UTF-8` and do not include credentials.

## Edit without coding (GTM JSON generator)

For a no-code way to adjust texts/styles and generate a ready-to-import GTM JSON container for this cookie banner, use the Databandits tool at `https://databandits.nl`.

## Contributing

Contributions are welcome! Please follow these guidelines:

1. Open an issue first to discuss significant changes.
2. Fork the repo and create a feature branch (`feat/your-change` or `fix/your-bug`).
3. Make focused commits with clear messages (e.g., Conventional Commits).
4. Keep changes scoped; avoid unrelated formatting churn. Preserve indentation and code style.
5. Add/update documentation in `README.md` when behavior changes.
6. Open a pull request and describe the change, motivation, and testing performed.

## License

Apache License 2.0. See [`LICENSE`](./LICENSE).
