---
name: vercel-react-native-skills
description:
  React Native and Expo best practices for building performant mobile apps. Use
  when building React Native components, optimizing list performance,
  implementing animations, or working with native modules. Triggers on tasks
  involving React Native, Expo, mobile performance, or native platform APIs.
license: MIT
metadata:
  author: vercel
  version: '1.0.0'
---
> **Fork notice**: originally installed from [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills). This version has been locally tuned; check the upstream repo for its license before redistributing.


# React Native Skills

Comprehensive best practices for React Native and Expo applications. Contains
rules across multiple categories covering performance, animations, UI patterns,
and platform-specific optimizations.

## When to Apply

Reference these guidelines when:

- Building React Native or Expo apps
- Optimizing list and scroll performance
- Implementing animations with Reanimated
- Working with images and media
- Configuring native modules or fonts
- Structuring monorepo projects with native dependencies

## Quick Reference

Rules grouped by impact (CRITICAL → LOW). Filename prefix matches section.

### 1. Core Rendering (CRITICAL)

- `rendering-text-in-text-component` - Wrap all strings in `<Text>` components
- `rendering-no-falsy-and` - Avoid `falsy && <JSX>` (renders "0" or crashes on Android)

### 2. List Performance (HIGH)

- `list-performance-virtualize` - Use LegendList (preferred) or FlashList; never `ScrollView` + `.map()`
- `list-performance-item-memo` - Memoize list item components; pass primitives, not objects
- `list-performance-callbacks` - Hoist callbacks out of `renderItem` to keep refs stable
- `list-performance-inline-objects` - No inline `{ style: {...} }` in `renderItem`
- `list-performance-function-references` - Define `renderItem`/`keyExtractor` outside render
- `list-performance-images` - Use compressed/sized images in list rows
- `list-performance-item-expensive` - Move expensive work (Intl, parsing) outside item render
- `list-performance-item-types` - Provide `getItemType` for heterogeneous lists

### 3. Animation (HIGH)

- `animation-gpu-properties` - Animate only `transform` and `opacity` (never width/height/top/left)
- `animation-derived-value` - Prefer `useDerivedValue` over `useAnimatedReaction`
- `animation-gesture-detector-press` - Use `GestureDetector` + `Gesture.Tap` for animated press

### 4. Scroll Performance (HIGH)

- `scroll-position-no-state` - Never track scroll offset in `useState`; use shared values

### 5. Navigation (HIGH)

- `navigation-native-navigators` - Use native stack and native tabs over JS navigators

### 6. React State (MEDIUM)

- `react-state-minimize` - Minimize state; derive values during render
- `react-state-dispatcher` - Use functional `setState` updates to avoid stale closures
- `react-state-fallback` - State represents user intent only, not server cache

### 7. State Architecture (MEDIUM)

- `state-ground-truth` - State must represent ground truth, not a copy

### 8. React Compiler (MEDIUM)

- `react-compiler-destructure-functions` - Destructure props/objects early for compiler memoization
- `react-compiler-reanimated-shared-values` - Use `.get()`/`.set()` for shared values under compiler

### 9. UI Patterns (MEDIUM)

- `ui-expo-image` - Use `expo-image` for all images
- `ui-image-gallery` - Use Galeria for lightbox/gallery
- `ui-pressable` - Use `Pressable` over `TouchableOpacity`
- `ui-safe-area-scroll` - Use `contentInsetAdjustmentBehavior` for safe areas
- `ui-scrollview-content-inset` - Use `contentInset` for dynamic header spacing
- `ui-menus` - Use Zeego native context menus
- `ui-native-modals` - Use native `Modal` with `formSheet`
- `ui-measure-views` - Use `onLayout`, not `measure()`
- `ui-styling` - Use `StyleSheet.create` or Nativewind; use `gap`, `boxShadow`, gradients

### 10. Design System (MEDIUM)

- `design-system-compound-components` - Use compound components for related UI parts

### 11. Monorepo (LOW)

- `monorepo-native-deps-in-app` - Install native deps in app package, not workspace root
- `monorepo-single-dependency-versions` - Single version per dependency across packages

### 12. Imports (LOW)

- `imports-design-system-folder` - Import from design system folder, not deep paths

### 13. JavaScript (LOW)

- `js-hoist-intl` - Hoist `Intl.NumberFormat`/`DateTimeFormat` creation outside render

### 14. Fonts (LOW)

- `fonts-config-plugin` - Load fonts natively at build time via config plugin

## How to Use

### Workflow

1. **Identify task type** from the user prompt (build feature, optimize perf, review code, fix bug).
2. **Look up applicable rules** in the Task → Rules table below.
3. **Read each referenced rule file** (`rules/<rule-id>.md`) before writing or critiquing code; each has Incorrect/Correct examples.
4. **Apply rules in priority order** (CRITICAL → HIGH → MEDIUM → LOW). Never skip CRITICAL.
5. **When in doubt or rule is ambiguous**, consult `AGENTS.md` (full compiled guide).

### Task → Rules Decision Table

| If user task involves...                          | Always check these rules                                                                                                                                                       |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Any list, feed, grid, table, results              | `list-performance-virtualize`, `list-performance-item-memo`, `list-performance-callbacks`, `list-performance-inline-objects`, `list-performance-function-references`, `list-performance-images` |
| Images (avatar, thumbnail, hero, gallery)         | `ui-expo-image`, `list-performance-images`, `ui-image-gallery` (lightbox)                                                                                                       |
| Animation, gesture, swipe, drag, transition       | `animation-gpu-properties`, `animation-derived-value`, `animation-gesture-detector-press`, `scroll-position-no-state`                                                           |
| Scroll tracking, parallax, sticky headers         | `scroll-position-no-state`, `ui-scrollview-content-inset`, `ui-safe-area-scroll`                                                                                                |
| Navigation, tabs, stack                           | `navigation-native-navigators`                                                                                                                                                  |
| Buttons, taps, press feedback                     | `ui-pressable`, `animation-gesture-detector-press`                                                                                                                              |
| Modals, sheets, dropdowns, menus                  | `ui-native-modals`, `ui-menus`                                                                                                                                                  |
| Text rendering, conditional rendering             | `rendering-text-in-text-component`, `rendering-no-falsy-and`                                                                                                                    |
| State, hooks, re-renders                          | `react-state-minimize`, `react-state-dispatcher`, `react-state-fallback`, `state-ground-truth`, `react-compiler-destructure-functions`, `react-compiler-reanimated-shared-values` |
| Styling, layout, theme                            | `ui-styling`, `list-performance-inline-objects`                                                                                                                                 |
| Reanimated shared values                          | `react-compiler-reanimated-shared-values`, `animation-derived-value`                                                                                                            |
| View dimensions, measuring                        | `ui-measure-views`                                                                                                                                                              |
| Custom fonts, typography                          | `fonts-config-plugin`                                                                                                                                                           |
| Monorepo setup, package layout                    | `monorepo-native-deps-in-app`, `monorepo-single-dependency-versions`                                                                                                            |
| Imports, design system structure                  | `imports-design-system-folder`, `design-system-compound-components`                                                                                                             |
| Internationalization, formatting                  | `js-hoist-intl`                                                                                                                                                                  |
| Code review / perf audit (no specific task)       | Scan all CRITICAL + HIGH rules first; flag any violations                                                                                                                       |

### Reading rule files

Each `rules/<id>.md` has:
- Frontmatter (`title`, `impact`, `tags`)
- One-paragraph rationale
- **Incorrect** code example
- **Correct** code example
- Reference link

### Fallback paths

- If the task does not map to any row above, scan CRITICAL rules first, then HIGH.
- If a rule file is missing or unclear, fall back to `AGENTS.md` (full compiled guide).
- If `AGENTS.md` is also missing, rely on the Quick Reference summaries here.

## Full Compiled Document

For the complete guide with all rules expanded: `AGENTS.md`
