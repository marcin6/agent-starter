# UI Prototype

Generate **several radically different UI variations** on a single route, switchable from a floating bottom bar. The user flips between variants in the running app on a simulator or a phone, picks one (or steals bits from each), then throws the rest away.

If the question is about logic/state rather than what something looks like — wrong branch. Use [LOGIC.md](LOGIC.md).

## When this is the right shape

- "What should this screen look like?"
- "I want to see a few options for this dashboard before committing."
- "Try a different layout for the settings screen."
- Any time the user would otherwise spend a day picking between three vague mockups in their head.

## Two sub-shapes — strongly prefer sub-shape A

A UI prototype is much easier to judge when it's **butting up against the rest of the app** — real header, real tab bar, real data, real density. A throwaway route on its own is a vacuum: every variant looks fine in isolation. Default to sub-shape A whenever there's a plausible existing screen to host the variants. Only reach for sub-shape B if the prototype genuinely has no nearby home.

### Sub-shape A — adjustment to an existing screen (preferred)

The route already exists. Variants are rendered **on the same route**, gated by a `variant` route param (read with `useLocalSearchParams`). The existing data fetching, params, and auth all stay — only the rendering swaps. This is the default; pick it unless there's a specific reason not to.

If the prototype is for something that doesn't yet have a screen but _would naturally live inside one_ (a new section of the home tab, a new card on the settings screen, a new step in an existing flow) — that's still sub-shape A. Mount the variants inside the host screen.

### Sub-shape B — a new screen (last resort)

Only use this when the thing being prototyped genuinely has no existing screen to live inside — e.g. an entirely new top-level surface, or a flow that can't be embedded anywhere sensible.

Create a **throwaway route** following whatever routing convention the project already uses — don't invent a new top-level structure. Name it so it's obviously a prototype (e.g. include the word `prototype` in the path or filename). Same `variant` param pattern.

Before committing to sub-shape B, sanity-check: is there really no existing screen this could be embedded in? An empty route hides design problems that a populated one would expose.

In both sub-shapes the floating bottom bar is identical.

## Process

### 1. State the question and pick N

Default to **3 variants**. More than 5 stops being radically different and starts being noise — cap there.

Write down the plan in one line, in the prototype's location or a top-of-file comment:

> "Three variants of the settings screen, switchable via the `variant` param, on the existing `/settings` route."

This works whether the user is here to push back or not.

### 2. Generate radically different variants

Draft each variant. Hold each one to:

- The screen's purpose and the data it has access to.
- The project's component library / styling system (`StyleSheet` with theme tokens, NativeWind, a UI kit, whatever).
- A clear exported component name, e.g. `VariantA`, `VariantB`, `VariantC`.
- A home outside `src/app` (every file there becomes a route), e.g. `src/components/<feature>/prototype/VariantA.tsx`.

Variants must be **structurally different** — different layout, different information hierarchy, different primary affordance, not just different colours. Three slightly-tweaked card lists isn't a UI prototype, it's wallpaper. If two drafts come out too similar, redo one with explicit "do not use a card list" guidance.

### 3. Wire them together

Create a single switcher component on the route:

```tsx
// pseudo-code — adapt to the project
const { variant = "A" } = useLocalSearchParams<{ variant?: string }>();
return (
  <View style={{ flex: 1 }}>
    {variant === "A" && <VariantA {...data} />}
    {variant === "B" && <VariantB {...data} />}
    {variant === "C" && <VariantC {...data} />}
    <PrototypeSwitcher variants={["A", "B", "C"]} current={variant} />
  </View>
);
```

For sub-shape A (existing screen): keep all the existing data fetching above the switcher; only the rendered subtree changes per variant.

For sub-shape B (new screen): the throwaway route under `src/app/prototype/<name>.tsx` mounts the same switcher.

### 4. Build the floating switcher

A small absolutely positioned bar at the bottom-centre of the screen, above the home indicator and tab bar (use the safe area insets), with three pieces:

- **Left arrow** (a `Pressable`) — cycles to the previous variant (wraps around).
- **Variant label** — shows the current variant key and, if the variant exports a name, that name too. e.g. `B — Sectioned list`.
- **Right arrow** (a `Pressable`) — cycles forward (wraps around).

Behaviour:

- Tapping an arrow updates the route param (`router.setParams({ variant })`) so the variant survives a reload and can be opened directly with a deep link (`<scheme>://settings?variant=B`).
- The bar stays clear of the keyboard and does not cover the screen's primary action; if it does, let it collapse to a small handle.
- Visually distinct from the screen (e.g. high-contrast pill, subtle shadow) so it's obviously not part of the design being evaluated.
- Hidden in production builds — render it only when `__DEV__` is true, so a stray prototype merge can't ship the bar to users.

Put the switcher in a single shared component so both sub-shapes can reuse it. Locate it wherever shared UI lives in the project.

### 5. Hand it over

Surface the route and the deep link for each variant (`<scheme>://settings?variant=A`, …), and say whether you checked them on iOS, Android or both. The user will flip through whenever they get to it. The interesting feedback is usually **"I want the header from B with the list from C"** — that's the actual design they want.

### 6. Capture the answer and clean up

Once a variant has won, write down which one and why (commit message, ADR, issue, or a `NOTES.md` next to the prototype if running AFK and the user hasn't responded yet). Then:

- **Sub-shape A** — delete the losing variants and the switcher; fold the winner into the existing screen.
- **Sub-shape B** — promote the winning variant to a real route, delete the throwaway route and the switcher.

Don't leave variant components or the switcher lying around. They rot fast and confuse the next reader.

## Anti-patterns

- **Variants that differ only in colour or copy.** That's a tweak, not a prototype. Real variants disagree about structure.
- **Sharing too much code between variants.** A shared `<Header>` is fine; a shared `<Layout>` defeats the point. Each variant should be free to throw out the layout.
- **Wiring variants to real mutations.** Read-only prototypes are fine. If a variant needs to mutate, point it at a stub — the question is "what should this look like", not "does the backend work".
- **Promoting the prototype directly to production.** The variant code was written under prototype constraints (no tests, minimal error handling). Rewrite it properly when you fold it in.
