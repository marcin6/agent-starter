---
name: frontend-design
description: "Use when implementing or reviewing pages, forms, product views, modals, controls, layout, or user interactions."
---

# Match the existing interface

Make the requested interface look as if it had always belonged to the app. Visual continuity, familiar behavior and reuse are the goal; a task is not a chance to invent a new aesthetic.

## Start with evidence

Before writing UI code:

1. Keep the requested scope intact.
2. Find the closest shipped page or flow in `src/app/` and its components in `src/components/` (a list, a detail page, a form, a static text page).
3. Search `src/ui/` for the building blocks, and read the component itself, not only its name.
4. Check the Tailwind config and `src/styles/index.css` for the theme.

Design from the repository, not from memory. When patterns conflict, follow the closest current screen and mention the conflict if it matters.

**A new project with no comparable screen yet:** agree on the theme first (font, brand colors as named Tailwind tokens, button and field shapes) and record it under "Look" below. The first screens set the pattern every later screen follows, so build their controls as `src/ui` components from the start.

## Building blocks

<!-- TEMPLATE: name the styling stack and list the real src/ui components with their props as they appear. -->

The UI is <Tailwind classes on native HTML elements and the app's own components in `src/ui`, with `clsx` for conditional classes; no UI library>. Pick each control in this order:

1. The component the closest comparable screen uses.
2. A component from `src/ui`: <`Button` (variants, sizes, `isLoading`), `ButtonLink`, `TextField`, `Loader`, …>.
3. A native element styled with Tailwind, following the closest screen: radio choices are a `fieldset` with a `legend` and labelled `input type="radio"`; a popup panel closes on Escape, on a click outside and after navigation.
4. Only then a new component in `src/ui/<component>/` with an `index.ts`, exported from `src/ui/index.ts`.

Adding a UI library is the developer's decision: propose it, never start it in a task.

## Look

<!-- TEMPLATE: fill in the project's theme. -->

- **Font:** <font family, where it is set>.
- **Colors:** the brand palette is <`BRAND_COLORS`> in the Tailwind config, used as Tailwind classes (<`text-brand-…` for body text, `hover:text-brand-…` for links, accents>); neutrals come from one Tailwind gray scale. Never write colors as arbitrary hex values (`bg-[#daade0]`), and do not add new colors, gradients or shadows; a new color is first agreed with the developer and then added to the palette.
- **Buttons and fields:** as the `src/ui` components render them (<radius, hover, pressed state>).
- **Layout:** spacing and sizes come from the Tailwind scale used nearby, not arbitrary pixel values.
- **Responsive, mobile first:** design and build the phone layout first. Unprefixed Tailwind classes describe the phone; `sm:`, `md:` and `lg:` add the larger layouts (`w-full lg:w-[120px]`, not `w-[120px] max-lg:w-full`). Images and media fill the phone width and shrink only from a breakpoint up; nothing important is hidden on the phone without a phone equivalent. Width-dependent differences are Tailwind classes, never a JS media query: the server cannot know the screen width, so a JS check renders one layout first and jumps after hydration.
- **Images:** <`next/image`, or plain `<img>` with the backend's resized formats and the reason>. Give every image its real dimensions or a sized container so the layout does not shift, `loading="lazy"` below the fold, high priority only for the LCP image, and a meaningful `alt`.
- **Motion:** keep to what exists and respect reduced motion (`motion-reduce:`).

## Behavior

- **Text:** in the UI language, matching the tone and terms of the surrounding pages. No text left in another language.
- **Forms:** labelled inputs with `autoComplete` where the browser can fill the value, a validation message under the affected field, the field's error cleared when it changes, entered values kept after an error. Format hints belong in the message or placeholder, never instead of a label.
- **States:** every screen that waits for data or a request has a loading state, an empty state and an error state the user can understand.
- **Busy buttons:** a button whose action is running blocks a second click and keeps its label and width; show a spinner, not a different text.
- **No jumping content:** what the user already sees does not move or resize when data or client-only state (from `localStorage`) arrives. Render client-only parts after mount behind a placeholder of the same size.
- **Accessibility:** real `<button>` and `<a>` elements, visible focus, keyboard-reachable controls, labels on inputs, `alt` on images.

## Restraint

Keep custom CSS small and local; prefer Tailwind classes. Leave surrounding UI, familiar labels and control positions as they are unless the request requires a change. When an existing pattern cannot meet a concrete usability or accessibility need, state the problem and make the smallest correction.

## Verify the result

1. Compare the result with the selected comparable screen.
2. Run the dev server and check the changed page at a phone width (375 px) first, then a desktop width, including loading, empty and error states, focus and overflow.
3. For flows that submit data or take payment, walk through them end to end with test keys.
4. Confirm that no `src/ui` component was bypassed by a raw equivalent without reason.

In the handoff, name the screens and components the change follows and any deliberate exception. If the browser check could not be done, say so.
