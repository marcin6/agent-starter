---
name: file-organization
description: "Use when creating, moving, naming, splitting, or reviewing files, folders, routes, imports, or shared modules."
---

# File Organization

<!-- TEMPLATE: the tables are the default layout for a Next.js App Router frontend and a separate backend. Adjust them to the project once, name real examples from the codebase as it grows, then delete this comment. -->

Use this skill before creating, moving, renaming or reviewing files. Follow the layout below; keep an existing local convention when a scoped change does not need a rename, and never rename files only for consistency.

## Where code goes

### Frontend (`frontend/`)

| Area | Path | What goes here |
| --- | --- | --- |
| Routes | `src/app/**` | App Router segments: `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `route.ts`. A page stays thin: it reads data and renders a component from `src/components`. |
| Shared route code | `src/app/(<group>)/` | Code several routes reuse without adding a URL segment (shared metadata builders, layouts). |
| Global client state | `src/app/context/<name>context/` | App-wide React contexts with their pure reducers, types and an `index.ts`. |
| Providers | `src/app/providers.tsx` | The provider tree the root layout renders. Add a provider here, never in a page. |
| Page and feature components | `src/components/<feature>/` | One folder per feature or page, holding its components and an `index.ts` barrel. |
| Reusable UI | `src/ui/<component>/` | Presentational building blocks any feature may use (`Button`, `TextField`, `Loader`), each with an `index.ts`, re-exported from `src/ui/index.ts`. |
| Backend access | `src/data-access/<operation>/` | One folder per read or write (`fetchProducts/`): `<operation>.ts`, `validators.ts` (Zod schemas and inferred types), `index.ts`. |
| Hooks | `src/hooks/` | App-wide hooks, one flat file each (`useIsMounted.ts`). A hook used by one component stays beside it. |
| Analytics | `src/data-layers/` | Tracking setup and event helpers with their types. |
| Shared helpers and types | `src/shared/` | Domain-free helpers (`formatPrice.ts`) and types several features use (`types.ts`). One file per helper, exported from `src/shared/index.ts`. |
| Styles | `src/styles/index.css`, `tailwind.config.*` | Global CSS and the theme (brand colors, fonts). |
| Static files | `public/`, `src/images/` | Files served as is in `public/`; images imported by components in `src/images/`. |
| Global types | `global.d.ts` | Ambient declarations (`window.dataLayer`, asset modules). |

### Backend (`backend/`)

| Area | Path | What goes here |
| --- | --- | --- |
| Schemas | `<path>` | Data model definitions. |
| API logic | `<path>` | Controllers, routes and services; business logic shared by controllers goes into a service. |
| Input validation | `<path>` | Schemas that parse request bodies before any logic runs. |
| Configuration | `config/` | Values from environment variables, never hard-coded secrets. |
| Generated code | `<path>` | Written by tools; never edit. |

## Naming

- Component files PascalCase and named after their component (`CartItem.tsx` exports `CartItem`). Parts of a feature take the feature as prefix (`CartItem`, `CartSummary`).
- Other TypeScript files camelCase (`fetchProducts.ts`, `formatPrice.ts`); hooks start with `use`.
- Folders under `src/components`, `src/ui`, `src/hooks` and `src/data-access` are camelCase (`homePage`, `fetchSingleProduct`); route folders follow the URL (kebab-case).
- Next.js reserved file names exactly (`page.tsx`, `layout.tsx`, `route.ts`, …) and segment syntax only for routing intent (`[slug]`, `(group)`).
- Type and constant naming follows `AGENTS.md`, "Code conventions".

## Barrels and imports

- Every folder in `src/components`, `src/ui`, `src/data-access` and `src/app/context` has an `index.ts` that names what callers outside the folder use (`export { Cart } from "./Cart";`, `export type { IProduct } from "./validators";`), never `export *`. Callers import from the folder (`@/data-access/fetchProducts`, `@/ui`), not from files inside it.
- Import across folders with the `@/` alias; relative imports only inside one folder.
- Server-only code (anything reading a secret or a non-`NEXT_PUBLIC_` variable) is never imported by a `"use client"` file, directly or through a barrel.

## Placement rules

- Colocate first: code used by one component stays in its folder. Move it to `src/ui`, `src/hooks` or `src/shared` when a second feature needs it, or from the start when it is clearly generic (a formatter, a breakpoint hook, a button).
- Before writing a helper, search `src/shared` and `src/hooks`; a domain-free helper goes to `src/shared` from its first use, not as a private copy beside a component.
- A new backend read or write is a new `src/data-access/<operation>/` folder, never a `fetch` written inside a component.
- Tests live in a `__tests__` folder beside the code they test.

## Exceptions

<!-- TEMPLATE: list deliberate departures from the rules above, so they are neither "fixed" nor copied. Delete the section while it is empty. -->

| File | Departure | Reason |
| --- | --- | --- |
| <path> | <the rule it breaks> | <why it stays that way> |

A listed exception stays as it is and never serves as a pattern for new code. A new exception is agreed with the developer and added here with its reason.
