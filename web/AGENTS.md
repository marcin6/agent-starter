<!--
TEMPLATE. Replace every <placeholder>, delete every section or line that does not apply, then delete this comment and the other HTML comments (they load into the agent's context on every turn).
Keep only what an agent cannot find by reading the code: unwritten conventions, the reasons behind choices, hazards, lists of places that must change together. Commands and versions the agent can read from package.json do not belong here.
-->

# <Project name>

<One paragraph: what the product is, who uses it, the language of its UI, whether it takes payments and through what.> The repository holds <the layout, e.g. "two independent npm projects with no workspace root (the root `package.json` holds only `npm run dev`, which starts both)" or "a single Next.js app">:

- `frontend/`: <framework and version, React version, styling, where the own UI components live, validation library>. Node <version>.
- `backend/`: <framework, CMS or API, database locally and in production, hosting>. Node <version>.

Verify before answering or changing anything: read this file, the code involved and its callers. The user's claims and the existing code are evidence to check, not proof. If a request conflicts with this file, say so and ask for confirmation before proceeding.

## Local skills

Skills live in `.agents/skills/` (`.claude/skills` links to it). Read the applicable one before working in its area: `implement` for any feature, fix or refactor; `code-review` for reviewing changes; `file-organization` before creating, moving or naming files; `frontend-design` for any UI work; `next-best-practices` for Next.js routing, rendering, data fetching and metadata. Before using or changing a framework API, check the guide for the installed version (for Next.js: `frontend/node_modules/next/dist/docs/`), because major versions differ. Generic library advice applies only to the installed version and never justifies a new dependency, cache, abstraction or unrelated refactor on its own.

## Golden rules

- Keep things simple. YAGNI unless told otherwise. <Who maintains the project, e.g. "a small product maintained by one or two developers">: minimize concepts, moving parts and what someone must know to change the code safely.
- Implement the smallest complete solution for the agreed behavior. Add flexibility only for a current requirement or an existing consumer.
- Investigate before changing anything. Search for an existing solution (component in `src/ui`, helper in `src/shared`, data-access operation, hook) before creating a new one.
- Reuse sound patterns. Existing patterns are not automatically correct: if one is brittle, unsafe or needlessly complex, explain the problem and prefer the simplest durable correction within scope. Keep unrelated improvements out of scope; report them instead.
- Use type safety: strict TypeScript, a runtime validator (Zod) for every response from the backend or an external API, typed props.
- Propose bold ideas when they would meaningfully help (a new library, a test setup, a restructuring), but propose them to the developer instead of implementing them.
- Be objective: verify claims, state trade-offs and blind spots, recommend the strongest option plainly.
- Before declaring work complete, verify it in proportion to its risk (see "Checks"). Never claim success without evidence.

## How the app works

<!-- One short paragraph per domain area: its source of truth, its flow, and what must change together. Write the reason when a choice looks odd. The paragraphs below are defaults for a Next.js App Router frontend over a separate backend; adapt or delete them. -->

**Data and content.** <The source of truth for each kind of data, e.g. "the CMS holds products, categories and pages">. Schemas are defined in <path>; <path to generated types> is generated, never edited by hand.

**Reading data.** The frontend reads the backend through `frontend/src/data-access/<operation>/`: `fetch` against `<PUBLIC_API_URL variable>` with `next: { revalidate: <seconds> }`. Each operation keeps its Zod schema and inferred type in `validators.ts` beside it and parses every response with it: fields the backend does not guarantee are `nullable`, and a list skips and logs an invalid entry instead of failing the whole page. A list reads every page of a paginated API. Pages are async server components that await these functions and pass the data down as props. The browser never calls these functions; client components receive only the fields they render. Server actions are public endpoints: add one only when a client component truly needs it and the developer agrees. Keep the `src/ui/index.ts` barrel free of server-only components: client components import it, so everything exported there ships to the browser.

**Client state.** <What is global (a context read with a hook, a pure reducer), what stays page-local, what persists in `localStorage` and under which key.> Persisted state is validated when read, and reading browser storage must not break hydration: components showing it wait for mount.

**Writes and payments.** <The flow: which endpoint the browser calls, what the backend does, where the user lands after success and failure, which webhook confirms the result.> **Every amount charged is computed on the server from stored data, never taken from the request body.** Prices in the browser are display only.

**Analytics and consent.** <Analytics tool, how it loads (only after cookie consent?), where event helpers live.> Push new events through those helpers, never to the global queue directly.

**SEO.** Every page sets its metadata (`metadata` or `generateMetadata`). <How public URLs are built, which old URLs redirect, any product feed or sitemap.> Changing a public URL breaks indexed links; confirm it with the developer first.

**Change together.** <Lists of places that must change in one change, e.g. "the product `type` enum: the backend schema, the Zod schema, `TProductType`, the category routes, the feed mapping". Adding a field to an entity likewise means the backend schema, the fetch's field list and the Zod schema.>

## Risk areas

`implement` and `code-review` check every item the change touches:

- **Money.** Amounts charged come only from server-side data; they go to the payment provider in minor units (`Math.round(price * 100)`) and in the agreed currency. Delivery fees, discounts and taxes live in the backend; the frontend only displays them.
- **Persisted client state.** Data in `localStorage` or cookies outlives a deployment: a changed shape must still read old data or discard it safely.
- **Data shape.** A new or renamed field changes the backend schema, the fetch and the validator together. A schema change alters the production database on deploy: state its effect on existing entries.
- **Server/client boundary.** Secrets and server-only code stay on the server; browser-only APIs run only on the client, without a hydration mismatch.
- **SEO and URLs.** New pages get metadata; public URLs keep working; feeds and sitemaps stay valid.
- **Analytics and consent.** Event names and payload shapes stay stable; tracking tags load only after consent.
- <Project-specific risk, e.g. "**Bookings.** A slot is reserved only after payment confirmation.">

## Security and configuration

- Every `NEXT_PUBLIC_*` variable is shipped to the browser. Secrets (payment keys, API tokens with write access, admin secrets) live only in the backend environment and never carry the `NEXT_PUBLIC_` prefix.
- `.env` files hold real keys: never print their values, never commit one, and add new variables to `.env.example` with placeholder values.
- Public (anonymous) access is granted per endpoint, only as far as the product needs it. Personal data (e-mail, address, phone) is never publicly readable and never logged.
- Validate every input that reaches a backend endpoint with a schema, answer errors without internal details, and rate limit public write endpoints. CORS admits only the app's own origins.
- Remote images come only from hosts listed in `images.remotePatterns` of `next.config`.
- Production data lives on <host>. Before a schema change or a backend upgrade, state the effect on existing data, confirm with the developer and make sure a backup exists (<backup command or "ask the developer">).

## Code conventions

<!-- Defaults; change them before the first line of code and keep them afterwards. -->

Follow `file-organization` for placement and `frontend-design` for UI. Project-wide:

- Imports across folders use the `@/` alias (`@/data-access/fetchProducts`, `@/ui`, `@/shared`); relative imports only inside one folder.
- Named exports. `export default` only where the framework reads it (`page`, `layout`, `route`, `loading`, `error`, `not-found`) and in config files.
- Naming: interfaces `I` + PascalCase (`IProduct`, a component's props as `I{Component}Props`), type aliases `T` + PascalCase (`TProductType`), Zod schemas `{Name}Schema`, module constants holding a fixed value `UPPER_SNAKE_CASE`. Names say what they hold, spelled out (`deliveryPrice`, not `res` or `data` where a domain name exists); booleans read as questions (`isLoading`, `hasDiscount`).
- New functions take one destructured, typed object (`fetchProduct({ id })`); callbacks and signatures dictated by a framework keep their shape.
- Asynchronous code uses `async`/`await` with `try`/`catch`, never `.then()` chains. A failure the user should know about is shown in the UI, not only logged.
- Components are server components by default. Add `"use client"` only to the component that needs state, effects, context, event handlers or browser APIs, as low in the tree as possible.
- Comments are in English, and only where a reader would otherwise be misled: a non-obvious reason, an external constraint, or a hazard. No comments that narrate the change or restate the code; no new commented-out code.
- User-facing text is <language>, <"written directly in the components (no translation layer)" or "kept in <i18n setup>">. Keep the established tone and wording of nearby screens. Route folders are <language> kebab-case.
- Prices and dates are formatted with the shared helpers in `@/shared` (<locale>).

## Checks

<!-- Put the real script names here once package.json exists. -->

`npm run dev` <in the root starts both projects / in `frontend/` starts the app>. Tests use Vitest (`npm test`; tests in `__tests__` folders beside the code). Logic where a mistake costs money or blocks users (prices, payments, input validation, persisted state) gets a test with every change. Lint passes with no errors or warnings, and every change keeps it that way.

- Frontend (`cd frontend`): `npm run type-check`, `npm run lint` and `npm test` pass on every change; run `npm run build` for changes to routing, metadata, server/client boundaries or configuration. For UI changes, run `npm run dev` and check the changed page in the browser at mobile and desktop widths.
- Backend (`cd backend`): `npm test` and `npx tsc --noEmit -p .` pass. A change to a payment flow is verified end to end against the provider's test keys, never live keys.
- Use npm and the existing `package-lock.json`. Install a new dependency only after the developer agrees to it. Install and upgrade only versions released at least two weeks ago (`npm install --before=<date>`); a fix for a critical, published vulnerability may be newer, and says so in the pull request. Check `npm audit` when upgrading.

## Deployment

- <Which branch deploys what where, e.g. "Every merge or push to `master` is a production release of the frontend (Vercel)">. Treat a pull request to that branch as a deployment and verify it before merging: type-check, build, and the preview deployment when one exists.
- Production environment variables live in <host> settings, not in the repository. A change that reads a new variable is merged only after the variable exists there; say so in the pull request.
- <Backend hosting.> Its deployment is separate from the frontend's; deploy backend changes the frontend depends on first, and ask the developer before relying on a backend change being live.

## Git

- Every change starts from `<integration branch>`: update it and create a kebab-case feature branch from it (`add-product-filters`).
- Pull requests go from the feature branch to `<release branch>`.
- Commit, push, open pull requests or merge only when the developer asks.
- Commit messages: a short, lowercase, imperative subject without a type prefix (`add product filters`).
