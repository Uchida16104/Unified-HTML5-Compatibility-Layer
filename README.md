# Unified HTML5 Compatibility Layer

An HTML5-first architecture prototype that demonstrates a shared component contract, a small component registry, an HTMX endpoint tester, and browser-side markup checks.

> **Scope:** This repository is a frontend architecture prototype, not a universal compiler. HTML5 and HTMX provide a shared interface. JavaScript, TypeScript, CSS preprocessors, PHP, and SQL continue to use their native runtimes and toolchains.

## Project structure

```text
unified-html5-compatibility-layer/
├── index.html   # Complete browser application: markup, styles, registry, validator, and UI logic
└── README.md    # Architecture overview and usage guide
```

## Features

- **HTML5-first component contract:** semantic elements, stable IDs, CSS classes, and documented `data-*` attributes.
- **Component registry:** maps `data-component` values to component responsibilities and adapter boundaries.
- **HTMX integration:** includes HTMX 2.0.11 and an endpoint tester using `hx-get`, `hx-target`, and `hx-swap`.
- **Browser-side markup checks:** checks duplicate IDs, registry values, runtime/storage labels, common HTMX target and swap attributes, and inline event-handler warnings.
- **Adapter catalog:** documents integration boundaries for JavaScript, TypeScript, CSS preprocessors, PHP, SQL, and HTMX.
- **Responsive UI:** includes a component browser, markup editor, validation report, runtime catalog, and architecture notes.
- **No project build step:** the prototype itself uses native HTML, CSS, and JavaScript.

## Requirements

- A modern browser with JavaScript enabled.
- Network access to the jsDelivr CDN for the HTMX library, or a locally hosted copy of HTMX.
- A web server and backend route only when using the HTMX endpoint tester.

The overview, registry, and markup validator work without a backend. A local static-file server is sufficient for those features.

## Run locally

From the project directory, start a static server. For example, with Python:

```bash
python3 -m http.server 8000
```

Then open:

```text
http://localhost:8000/
```

You can also open `index.html` directly in a browser. A local HTTP server is recommended when testing browser security behavior and HTMX requests.

## HTMX endpoint tester

The Runtime adapters page contains a button configured with:

```html
<button
  id="htmx-request"
  type="button"
  hx-get="/api/health"
  hx-target="#htmx-response"
  hx-swap="innerHTML"
>
  Send HTMX request
</button>
```

The prototype does **not** include `/api/health`. Connect a backend route before using this button. The route should return an HTML fragment suitable for insertion into `#htmx-response`, for example:

```html
<p role="status">Backend is available.</p>
```

For another route, change the `hx-get` value in `index.html`. The backend must implement the corresponding HTTP method, authentication, authorization, input validation, and response behavior.

## Component contract

The example uses the following attributes:

| Attribute | Purpose | Example |
|---|---|---|
| `data-component` | Registry key | `data-component="application"` |
| `data-runtime` | Runtime category | `data-runtime="javascript"` |
| `data-action` | Action identifier | `data-action="fetch-status"` |
| `data-storage` | Storage category | `data-storage="local"` |
| `data-api` | API route/URL metadata | `data-api="/api/status"` |
| `data-validator` | Validation profile identifier | `data-validator="required,email"` |
| `data-animation` | Animation profile identifier | `data-animation="fade"` |

Only attributes actually needed by a component should be added. Keep values documented and validated. IDs must remain unique within the document. Use classes for styling, IDs for unique targets, and ARIA attributes for accessibility.

### Registered component keys

| `data-component` | Responsibility |
|---|---|
| `application` | Application initialization boundary |
| `action` | User-triggered action |
| `result` | Result or response region |
| `form` | Form boundary |
| `field` | Input field and its validation metadata |
| `status` | Status and asynchronous feedback |

To add a component, update the `COMPONENTS` array in `index.html`. If it introduces new runtime or storage values, update the corresponding allowlist near the registry definition and add validation rules as appropriate.

## Adapter contract proposal

A production implementation can standardize a small lifecycle interface while allowing each framework to implement it natively:

```ts
export interface ComponentConfig {
  component: string;
  runtime?: string;
  options?: Record<string, unknown>;
}

export interface ComponentAdapter {
  mount(element: HTMLElement, config: ComponentConfig): void | Promise<void>;
  destroy?(element: HTMLElement): void | Promise<void>;
}
```

The component registry can then associate a component key with one or more adapters. For example, a Vue adapter, Alpine adapter, and plain JavaScript adapter may mount the same stable HTML contract, but they remain separate implementations and should not all initialize the same element unless explicitly coordinated.

## Technology boundaries

| Technology | Appropriate responsibility | Native validation/build tools |
|---|---|---|
| HTML5 | Document structure, semantics, accessibility | Nu HTML Checker / browser tests |
| HTMX | HTTP interactions and HTML fragment swaps | Endpoint integration and browser tests |
| JavaScript | Browser behavior and Node.js application logic | ESLint, unit tests, runtime tests |
| TypeScript | Static type system and generated declarations | `tsc --noEmit`, type tests |
| Tailwind CSS / Bootstrap / EaseMotion CSS | Utility and component styling systems | Framework build and visual tests |
| SCSS / SASS / LESS | Stylesheet preprocessing | The selected preprocessor compiler |
| PHP | Server-side routes and application logic | `php -l`, framework and integration tests |
| SQL | Persistence, queries, migrations, transactions | The configured database engine and migration tests |
| sql.js | SQLite-compatible database execution in a browser/JavaScript environment | sql.js runtime tests |
| php.js | PHP-like helper functions implemented in JavaScript; not a PHP interpreter | Library-specific unit tests |
| PyScript | Python execution in supported browser contexts | PyScript/browser integration tests |
| Motion One | JavaScript animation runtime | Browser animation and accessibility tests |
| Next.js / Nuxt.js / Vue.js | Framework-specific application composition | Each framework's own build and test commands |
| PyScript | Python execution in supported browser contexts | PyScript/browser integration tests |

The HTML document does not compile TypeScript, preprocess SCSS/SASS/LESS, lint PHP, or execute arbitrary SQL. Those steps belong in a real build/CI pipeline.

## Markup validator scope

The built-in validator is intentionally a lightweight browser-side check. It currently checks:

- Empty input and presence of elements.
- Duplicate `id` values.
- Unknown `data-component` values.
- Unsupported `data-runtime` and `data-storage` labels.
- Common `hx-target` selector references and core `hx-swap` values.
- Empty HTMX request endpoint attributes.
- Ambiguous `data-api` references.
- Inline `on*` event handler attributes.
- Unknown HTMX attributes against a small core allowlist.

It does **not** provide complete HTML conformance validation. Browser HTML parsers recover from malformed markup, and DOM parsing alone is not a standards validator. It also does not verify that a server route exists or that a selector will resolve after a future HTMX swap. Use the Nu HTML Checker and integration tests for stronger coverage.

## Suggested CI pipeline

A real project should run the relevant tools conditionally based on the technologies it actually uses. An example pipeline sequence is:

1. HTML conformance and accessibility checks.
2. JavaScript linting and unit tests.
3. TypeScript type checking.
4. CSS linting and SCSS/SASS/LESS compilation.
5. Framework production builds.
6. PHP syntax checks and backend tests.
7. SQL migration validation and database integration tests.
8. Browser end-to-end and visual regression tests.

Do not report a technology as validated when its compiler, interpreter, or database engine was not run.

## Security and deployment notes

- Treat all `data-*` values as untrusted configuration until validated.
- Validate and authorize every backend request on the server; client-side checks are not security controls.
- Use parameterized SQL queries. Never build SQL by concatenating user-controlled input.
- Apply CSRF protection where the authentication model requires it.
- Return only the HTML fragment intended for an HTMX target; do not inject untrusted HTML.
- Prefer self-hosting pinned third-party assets in production and review dependency updates.
- Configure an appropriate Content Security Policy and other security headers at the server/CDN layer.

## HTMX dependency

This prototype references HTMX 2.0.11 from jsDelivr with Subresource Integrity. The version and integrity hash are based on the official HTMX installation documentation:

- <https://htmx.org/docs/>

For production, consider downloading the reviewed HTMX distribution and self-hosting it rather than depending on a third-party CDN at runtime.

## License

No license is specified. Add a `LICENSE` file before redistributing this project under an open-source license.
