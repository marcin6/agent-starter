# Lists and performance

## Measure first, in a release build

Development mode is several times slower than production (dev checks, no minification). Judge performance only in a release build on a real device, preferably a mid-range Android phone: `npx expo run:android --variant release`, `npx expo run:ios --configuration Release`, or a `preview` build. Profile with React Native DevTools (the profiler) and the platform tools before changing code; fix the measured cause.

## Lists

- Data that can grow uses `FlatList`, `SectionList` or <`FlashList`>, never `.map()` in a `ScrollView`.
- `keyExtractor` returns a stable ID, never the index.
- `renderItem` renders a memoized item component (`React.memo`) that receives primitives or stable objects; callbacks passed to items are stable (`useCallback`) or take the item ID.
- Give items a fixed or predictable height; with `FlatList` and fixed heights, `getItemLayout` removes measuring.
- Paginate with `onEndReached` and a query that loads pages (`useInfiniteQuery`), with a footer loader and a guard against loading the same page twice.
- Nested scrolling in the same direction (a vertical list inside a vertical `ScrollView`) breaks virtualization: use the list's header and footer components instead.

## Rendering

- Keep state as low as possible: state at the top of a screen re-renders the whole screen on every change.
- Context values that change often re-render every consumer: split the context, or move the frequently changing value into a store with selectors.
- <If the React Compiler is enabled: manual `useMemo` / `useCallback` / `memo` is rarely needed; follow the compiler's rules instead of adding them.>
- Avoid inline object and array literals as props to memoized children in hot paths.

## The UI thread

- Animations and gesture-driven motion run on the UI thread: Reanimated shared values and worklets, or `Animated` with `useNativeDriver: true`. Animating layout from JS state drops frames when the JS thread is busy.
- Long synchronous work (parsing a large payload, sorting thousands of items) blocks touches. Move it out of render, split it, or defer it until after the interaction finishes.

## Images

- `expo-image` for remote and local images: it caches on disk and in memory. Request a size close to the rendered size from the API or CDN; full-resolution photos in thumbnails cost memory and scroll performance.
- In lists, set `recyclingKey` (or the equivalent for the installed version) so recycled cells do not flash the previous image.

## Startup

- Keep the splash screen up (`SplashScreen.preventAutoHideAsync`) only until fonts and the session are ready, then hide it. Everything else loads after the first screen renders.
- Avoid heavy imports and work at module top level in files the root layout imports; they run before the first frame.
- Remove `console.log` from hot paths; logging large objects is slow on device.
