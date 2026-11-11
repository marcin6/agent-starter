---
name: mobile-design
description: "Use when implementing or reviewing screens, forms, lists, modals, navigation, controls, gestures, animations, or user interactions."
---

# Match the existing interface

Make the requested interface look as if it had always belonged to the app, and feel native on each platform. Visual continuity, familiar behavior and reuse are the goal; a task is not a chance to invent a new aesthetic.

## Start with evidence

Before writing UI code:

1. Keep the requested scope intact.
2. Find the closest shipped screen or flow in `src/app/` and its components in `src/components/` (a list, a detail screen, a form, a settings screen).
3. Search `src/ui/` for the building blocks, and read the component itself, not only its name.
4. Read `src/theme` for colors, spacing, radii and typography.

Design from the repository, not from memory. When patterns conflict, follow the closest current screen and mention the conflict if it matters.

**A new project with no comparable screen yet:** agree on the theme first (font, brand colors for light and dark mode as named tokens, spacing scale, button and field shapes, navigation style: tabs, stack, drawer) and record it under "Look" below. The first screens set the pattern every later screen follows, so build their controls as `src/ui` components from the start.

## Building blocks

<!-- TEMPLATE: name the styling stack and list the real src/ui components with their props as they appear. -->

The UI is <React Native core components styled with `StyleSheet` and `src/theme` tokens / NativeWind classes>, with the app's own components in `src/ui`; <no UI kit / the UI kit name>. Pick each control in this order:

1. The component the closest comparable screen uses.
2. A component from `src/ui`: <`Screen` (safe area, scroll, keyboard), `Button` (variants, sizes, `isLoading`), `TextField`, `ListItem`, `EmptyState`, `ErrorState`, …>.
3. A native control or platform component, following the closest screen: `Pressable` for touchables, `Switch`, the platform date picker, the native header from the navigator, `Alert` or an action sheet for confirmations, a stack screen with `presentation: "modal"` or a sheet for modals.
4. Only then a new component in `src/ui/<component>/` with an `index.ts`, exported from `src/ui/index.ts`.

Adding a UI kit, an animation library or a gesture library is the developer's decision: propose it, never start it in a task.

## Look

<!-- TEMPLATE: fill in the project's theme. -->

- **Font:** <font family, loaded with `expo-font` in the root layout before the splash screen hides / the system font>.
- **Colors:** every color is a token in `src/theme` with a light and a dark value (<`colors.text`, `colors.background`, `colors.primary`, …>), read through <`useTheme()`>. Never write colors as literal hex values in components, and do not add new colors, gradients or shadows; a new color is first agreed with the developer and then added to the theme. Shadows differ per platform (`shadow*` on iOS, `elevation` on Android, or `boxShadow` where supported): use the theme's shadow tokens.
- **Spacing and sizes:** from the theme scale (<`spacing.xs` … `spacing.xl`>), not arbitrary numbers. Touch targets are at least 44 × 44 pt (iOS) / 48 × 48 dp (Android); use `hitSlop` when the visible control is smaller.
- **Buttons and fields:** as the `src/ui` components render them (<radius, pressed state, disabled state>). Pressed feedback is visible on both platforms (opacity or highlight; `android_ripple` where the app uses it).
- **Safe areas:** every screen respects the notch, the home indicator and the Android navigation bar through `react-native-safe-area-context` (or the `Screen` component), never fixed paddings. Edge-to-edge on Android: content draws under the system bars, so insets apply there too.
- **Platform conventions:** navigation headers, back gestures, modals, pickers and confirmation dialogs follow each platform. Copy a pattern from one platform to the other only when the app already does it.
- **Images:** `expo-image` with real dimensions or a sized container, a placeholder or blurhash while loading, `contentFit` chosen deliberately, and an `accessibilityLabel` when the image carries meaning.
- **Motion:** keep to what exists; animations run on the UI thread (Reanimated or native-driven `Animated`) and respect reduced motion (`useReducedMotion` / `AccessibilityInfo`). Haptics only where the app already uses them.

## Behavior

- **Text:** in the UI language, matching the tone and terms of the surrounding screens. No text left in another language. Text scales with the system font size: give text containers flexible heights, and cap scaling (`maxFontSizeMultiplier`) only where layout truly cannot grow.
- **Forms:** a visible label for each input; `keyboardType`, `autoComplete` / `textContentType`, `autoCapitalize`, `secureTextEntry` and `returnKeyType` set so the keyboard and autofill fit the field; "next" moves focus to the next field and the last one submits. The focused input stays visible above the keyboard (<`KeyboardAvoidingView` / `react-native-keyboard-controller`>); tapping outside dismisses it. A validation message under the affected field, the field's error cleared when it changes, entered values kept after an error.
- **States:** every screen that waits for data or a request has a loading state, an empty state, an error state with a retry, and an offline state the user can understand. Lists support pull to refresh; screens show cached data while refetching instead of a spinner over everything.
- **Lists:** anything that can grow is a virtualized list (`FlatList` / <`FlashList`>), never `.map()` inside a `ScrollView`. Items have stable keys and fixed or predictable heights.
- **Busy buttons:** a button whose action is running blocks a second tap and keeps its label and width; show a spinner, not a different text.
- **No jumping content:** what the user already sees does not move or resize when data, fonts or stored state arrive. Keep the splash screen up until fonts and the initial session are ready; then render placeholders of the final size.
- **Navigation:** the Android back button and the iOS swipe-back go where the user expects; a screen with unsaved input asks before it is left; a screen opened from a deep link has a way back (to the tab root when there is no previous screen).
- **Accessibility:** every touchable has `accessibilityRole` and an `accessibilityLabel` when its content is not text; state is exposed (`accessibilityState` for selected, disabled, checked); reading order follows the visual order; nothing important is conveyed by color alone; contrast holds in both themes. VoiceOver and TalkBack reach every control.

## Restraint

Keep styles local to the component and built from theme tokens. Leave surrounding UI, familiar labels and control positions as they are unless the request requires a change. When an existing pattern cannot meet a concrete usability or accessibility need, state the problem and make the smallest correction.

## Verify the result

1. Compare the result with the selected comparable screen.
2. Run the app and check the changed screen on the iOS simulator and the Android emulator: a small phone first (iPhone SE size, a 5" Android), then a large one; light and dark mode; the largest common font size; with the keyboard open; in loading, empty, error and offline states; landscape only if the app supports it.
3. Turn on VoiceOver or TalkBack for new controls and check they are announced and reachable.
4. For flows that submit data or take payment, walk through them end to end with test accounts and sandbox purchases.
5. Confirm that no `src/ui` component was bypassed by a raw equivalent without reason.

In the handoff, name the screens and components the change follows, the platforms and sizes checked, and any deliberate exception. If a platform check could not be done, say so.
