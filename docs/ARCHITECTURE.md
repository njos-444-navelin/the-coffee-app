# Architecture and stack

The technical decisions for the project and the rules that follow from them. `MANIFESTO.md` is binding above everything; `docs/CONCEPT.md` describes the product; `docs/RECOMMENDATIONS.md` describes how recommendations work. Settled 2026-09-17; change by editing this file with a note in the commit message saying why.

## Stack

| Layer | Choice | Why, in one line |
|---|---|---|
| Platform | A mobile-first PWA. Store listings are a possible future by wrapping the same bundle in a native shell, not a design constraint | The product is a URL; fastest iteration; the QR scheme resolves through any camera app |
| Language | TypeScript everywhere | One toolchain; frontend contributors can touch the backend |
| Runtime | Node, pnpm | Foundation-governed, every library targets it first; runtimes are never mixed |
| API | Hono with Zod schemas; OpenAPI generated from them; typed client generated for the frontend | Small, standards-based, runtime-agnostic; schemas give compile-time types and runtime validation at the boundary |
| Database | Postgres with pgvector, PostGIS, pg_trgm; full-text search in Postgres | Relational entity graph, vectors, geography, and fuzzy matching in one self-hostable box |
| Data access | Drizzle ORM | SQL stays visible; tables in TypeScript; custom column types cover the extensions |
| Frontend | Svelte 5 with SvelteKit; Node adapter (a static adapter build stays possible for a native shell) | Components run once, which suits an app full of imperative objects (map, camera, cache) |
| Styling | Svelte scoped styles; design tokens in the W3C Design Tokens format compiled to CSS custom properties | Own design system; the tokens file is the contract between design and code |
| UI primitives | Platform first; a headless library (Bits UI, Melt UI, or Ark UI, to be chosen) only for the hard widgets | Keep platform semantics and keyboard behaviour; accessibility is mandatory |
| Maps | MapLibre GL behind a thin adapter (pins, clustering, pan/zoom, geocode) | Tile and geocoding providers swap; provider IDs never become keys |
| Offline | On-device cache of the person's own data plus a write outbox | Diary works without signal; new entries never conflict |
| Background jobs | A worker process reading a queue in Postgres (pg-boss, to be confirmed) | No extra service to run |
| Repository | One repository, pnpm workspaces, no task runner until builds are slow (then Turborepo) | Shared code is a folder; one command to set up |
| Hosting | Docker Compose locally; one VPS with a self-hosted deploy dashboard (Dokploy or Coolify); a CDN in front; nightly database dump to object storage | Cost of a coffee subscription; the container is portable to any host |
| CI | GitHub Actions: lint, typecheck, test, build, DCO check; automated dependency updates | |
| Observability | Self-hosted error tracking; no analytics SDKs | Manifesto 7 and 9 |

Still to choose, with the current lean: auth (Better Auth), i18n library (Paraglide), object storage (any S3-compatible, MinIO locally), component workshop (Storybook or an in-app route), map tiles (MapTiler), error tracking (Sentry self-hosted or GlitchTip), email (Resend or plain SMTP), tests (Vitest and Playwright). Ask before assuming any of them. The native shell (Capacitor or Tauri) is deliberately undecided until a store listing is wanted.

## Shape

- One repository: `apps/web` (SvelteKit), `apps/api` (Hono; modules per domain; web and worker entry points), `packages/schemas` (Zod, OpenAPI, generated client), `packages/tokens` (design tokens to CSS variables), `packages/ui` (design system components), `docs/`. Two named future exceptions: the identifier scheme specification (a standard under its own license) and large bulk-import dataset proposals may get their own repositories.
- Server code is a modular monolith: one module per domain (auth, catalog, diary, taste, wiki, events, jobs), each owning its tables, rules, routes, and jobs, reachable by other modules only through an explicit interface. Split into a service only for a named reason: scaling profile, language, owner, release cadence, or failure isolation.
- Two server processes from the same build. **web** is one Node program hosting two handlers: Hono answers requests under the API path; SvelteKit's adapter handler, mounted for every other path, renders public pages and serves the frontend bundle. Hono does no rendering; SvelteKit does no API work. **worker** has no HTTP and runs background jobs from the queue: embeddings, descriptor extraction, taste recomputation, summaries, photo processing, exports, notifications, housekeeping. Locally both may run in one process.
- SvelteKit code runs in two places: its rendering of public pages runs inside the web process; the app itself runs on the person's device, in the browser or later inside a native shell, as the bundle the web process serves.

## Client rules

Rules 1 and 2 are required by the public API and by offline; they also happen to keep a native shell possible.

1. **The API is the boundary.** The UI must run as a static bundle talking to the HTTP API. Nothing the logged-in app needs may depend on server-only rendering. In SvelteKit: data comes from the API through universal load functions or fetch, never through server-only load functions or form actions. The SvelteKit server exists to render public pages for search engines and link previews. (During server rendering, a relative fetch to the API is routed in-process through the handle hook; no network hop.)
2. **The API base URL is configuration.** Inside a native shell the page origin is not the product domain.
3. **Sessions use cookies.** Nothing else may assume cookies are the only possible session mechanism; if a native shell ever needs a different one, it is added then. Deep-link association files for a shell are two static files on the domain, added only if a shell exists.
4. **Offline:** the person's own data and anything they have viewed is cached on the device and shown immediately, refreshed when online. Every write, not only diary entries, goes through a local outbox and is delivered when a connection exists. Edits are versioned so that a conflict between two devices is surfaced to the person rather than resolved by overwriting. Downloadable map regions are a later feature.
5. **Platform first:** native elements and platform APIs, restyled; before building a non-native pattern because it would look good, look for a platform-native way to do the job. Headless primitives only for combobox, menu, tabs, listbox, and layered focus. Every component works by keyboard and screen reader.
6. **Styles:** scoped component styles referencing token-generated CSS custom properties only; no utility framework.
7. **One call per screen** where possible. Round trips, not hosting location, decide how the app feels far from the server.

## Data rules

1. **Identifiers:** client-generated, time-ordered UUIDs (v7) as primary keys on every table; retries are therefore idempotent. Public human-readable IDs for coffees, roasters, cafés, and farms are separate, permanent, and language-neutral (`docs/CONCEPT.md`, Identifiers).
2. **Every row has `created_at` and `updated_at`.** Deletions are soft (tombstones). Exports and the API respect tombstones. Together with client IDs, these are exactly what a sync engine would need, so that door stays open at zero cost.
3. **Provenance on every reference record:** created by, source, license, verification status (manifesto 3).
4. **Names are per-language lists,** not single strings. Canonical descriptors are translated; free descriptors are kept verbatim in their original language.
5. **Units are stored metric** and converted on display.
6. **Personal data and reference data are separable by schema,** so the open-data export and the per-user export are plain queries (manifesto 3 and 9).
7. **Embeddings carry the model name and version** that produced them; switching models is a background re-embed job, never a silent mix.

## API rules

- REST, described by OpenAPI. The identifier resolution endpoints are part of the published spec.
- Writes are idempotent (client-supplied IDs) and versioned (client sends the row version an update was based on).
- Public read API for reference data; personal data only through authenticated, owner-scoped endpoints.
- Identifier URLs (`/c/<id>` and similar) serve HTML to browsers and JSON to API clients on the same path.

## AI and model use

- All embedding and language-model calls go through an internal provider adapter (one interface, one implementation per provider, including a local one) so self-hosters can run without external API keys.
- Language models do per-record, cached jobs only, listed in `docs/RECOMMENDATIONS.md`. They never select recommendation candidates and never measure distance.
- Diary text sent to an external provider is the minimum needed, never accompanied by identity, only to providers whose terms exclude training on submitted data, and this is stated on the privacy page (manifesto 9).

## Internationalisation

- UI strings live in per-language message files with plural and ordering support, loaded one language at a time.
- Formatting (dates, numbers, units) uses the platform `Intl` APIs.
- Data is multilingual by the data rules above.
- English plus one other language from the first screen, to keep hardcoded strings out.

## Scaling path

In order, each only when measured: a bigger VPS; the worker on its own machine; managed Postgres or a read replica; a second region with a read replica only if a large, distant user base is not served well enough by the CDN and the on-device cache.
