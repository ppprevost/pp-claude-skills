---
name: pp-frontend-developer
description: Frontend stack opinions, duplication detection patterns, error boundaries, forms strategy, testing, and component patterns for React/Next.js projects following the pp-frontend-developer agent conventions. Use when reviewing or building React/TypeScript frontend code in projects using Zustand, TanStack Query, React Hook Form + Zod, Tailwind, Vitest, MSW, Radix, or Next.js App Router. Triggers on duplication, design system, component library, error boundary, Suspense, form strategy, state management, testing setup, accessibility.
license: MIT
metadata:
  author: pp-frontend-developer agent
  version: "1.2.0"
  triggers:
    - duplication detected
    - design system
    - component library
    - error boundary
    - Suspense
    - form strategy
    - state management
    - testing setup
    - accessibility
    - tailwind tokens
    - zustand
    - tanstack query
    - react hook form
    - vitest
    - msw
    - radix
    - compound component
---

# pp-frontend-developer Skill

Stack-opinionated frontend conventions + reusable patterns. Companion to `pp-frontend-developer` agent (workflow). Provides knowledge the agent defers here.

## TL;DR

```
review/build task → match a row in [Task → Section]
                  → scan code for [Code-Pattern Triggers]
                  → apply pattern from catalog
                  → cite the pattern (`compound component`, `Result<T,E>`, `Suspense+ErrorBoundary`, ...)
```

Skip when: not React/TS frontend (use `nodejs-backend-patterns`), pure type question (use `mastering-typescript`), pure perf question (use `vercel-react-best-practices`).

## Workflow

0. **Check for an existing solution FIRST.** Before writing any custom code, verify whether a library or primitive already solves the problem: a design-system primitive (`Heading`/`Text`/`Stack`/`Button`/`Card`...), a Radix primitive, an installed dependency, or a maintained well-known lib. If one exists, use it — do not hand-roll. Only build custom when nothing fits, and say why.
1. **Classify** task via Task → Section table.
2. **Read** the relevant section(s) below.
3. **Detect** code-pattern triggers; map to fix.
4. **Apply** the pattern. Default to opinionated stack choices unless project overrides.
5. **Cite** the pattern in diff/comment.

### Non-negotiable: no raw DOM elements with inline styles

Never emit raw `<div>` / `<p>` / `<h1>`-`<h6>` / `<span>` styled inline. Map them to design-system primitives:
- titles → `<Heading level size tone />`
- body/labels → `<Text variant as />`
- layout (`flex flex-col gap-N`, `flex items-center gap-N`) → `<Stack direction gap align justify />`
- repeated styled boxes / panels → an existing component or a new shared primitive

Raw `<div>` stays allowed only for pure centering (`flex items-center justify-center`, no gap) and non-layout className (rounded/padding/bg). Interactive elements use the real component (`<Button>`, Radix), never a styled `<div>`.

## Task → Section Table

| User intent | Section |
|---|---|
| "extract repeated JSX / inline style / helper" | Duplication Catalog |
| "should I use Zustand or Context?" | State Management |
| "global state library" | State Management |
| "API/server data fetching strategy" | Data Fetching |
| "form: RHF, Server Action, validation?" | Forms |
| "error handling, ErrorBoundary, Suspense" | Error Handling |
| "tests: Vitest, MSW, mocking" | Testing |
| "design tokens, Tailwind, colors, theme" | Styling |
| "accessibility, ARIA, keyboard nav" | Accessibility |
| "memoization, useMemo, useCallback, React 19" | Performance |
| "compound component, headless, primitives" | Component Patterns |
| "code splitting, lazy load, virtualization" | Performance |
| "monorepo types, codegen, shared packages" | TypeScript Boundaries |
| "where does this go / layering / feature structure / component imports infra" | Feature-First DDD Layering |

## Code-Pattern Triggers

| Pattern in code | Action |
|---|---|
| `style={{ background: 'rgba(...)', color: '#...' }}` repeated 2+ files | extract `<GlassPanel>` or design token CSS var |
| `<div className="flex items-center gap-X"><Icon/><h2>...</h2></div>` 3+ pages | extract `<SectionHeader>` |
| Raw `<h1>`-`<h6>` / `<p>` / `<span>` with inline styles | use `<Heading>` / `<Text>` design-system primitives |
| `<div className="flex flex-col gap-N">` or `flex items-center gap-N` | use `<Stack direction gap />` primitive |
| Hand-rolling something a library/primitive already provides (dropdown, dialog, date picker, carousel, tooltip) | check installed deps + design system FIRST, use the existing one |
| Component / hook in `use-cases/` importing from `infrastructure/` directly | inward deps only — go through a `presentation/actions/` wrapper or a use-case; see Feature-First DDD Layering |
| `'use server'` action doing business logic inline (not a thin wrapper) | actions = auth + call use-case + error mapping; logic lives in `use-cases/` |
| Domain/presentation type imported from another feature's internals | cross-feature only at `presentation/`, or go through an action/API; never reach into another feature's `use-cases`/`infrastructure` |
| `const showToast = ...` redeclared in N files | use shared `toast.success()` from design system |
| `useState<{loading, data, error}>` | use discriminated union `{status: 'idle'\|'loading'\|...}` |
| `try/catch` in component for async error | move to ErrorBoundary upstream |
| `useMemo` on trivial computation | drop, React Compiler handles it |
| `<select>` rolled by hand | use Radix `<Select>` primitive |
| `useState` global counter shared across components | move to Zustand store |
| `fetch` direct in component | move to TanStack Query `useQuery` (or RSC) |
| Custom validation logic | replace with Zod schema |
| `class App extends Component` | rewrite as function component, justify if class needed |
| `interface Foo` in non-merging context | use `type Foo = {}` |
| `forEach` with side-effect mutation | rewrite as `map`/`filter`/`reduce` |
| `switch/case` on union | replace with `Record<K, V>` + `satisfies` |
| Same `<View style={styles.row}>` 20+ in React-PDF doc | extract `<Row label value />` component |
| Same backtick template `\`<p>${a}: ${b}</p>\`` 3+ times (email/PDF/CSV) | extract `renderField(label, value)` + iterate |
| `useEffect(() => fetch(...), [])` for data fetch | use TanStack Query `useQuery` (SPA) or RSC (Next.js) |
| `Sentry.captureException(err)` in component try/catch | move to ErrorBoundary upstream, Sentry = monitoring layer not error strategy |
| `import 'styled-components'` / `@emotion/...` in new code | use Tailwind + design tokens unless legacy project |
| `import { Provider } from 'react-redux'` in new module | migrate to Zustand for client state, TanStack Query for server state |
| `<select>` / `<dialog>` rolled by hand for accessibility | use Radix `<Select.Root>` / `<Dialog.Root>` |
| `useMemo(() => a + b, [a, b])` trivial | drop, React Compiler (React 19) handles it |
| Manual `setLoading(true)` + `setLoading(false)` flag | use Suspense + `<Skeleton />` fallback |
| `while (true)` / `for (;;)` with `if (...) break` as real exit | rewrite with an explicit loop condition. For stream readers use read-ahead: `let chunk = await reader.read(); while (!chunk.done) { ...; chunk = await reader.read() }`. Reserve infinite-loop syntax for genuinely unbounded loops (event pumps) and comment why |

## Duplication Catalog

Patterns à factoriser (par ordre d'application).

### 1. Helper de rendu (le plus léger)

Si même fragment JSX revient :
```ts
const simpleAnswer = (key: string) => <p className="text-sm leading-relaxed">{t(key)}</p>
```
Plus léger qu'un composant, lisible inline.

### 2. Data-driven

Séparer donnée et structure :
```ts
const KEYS = ['q1', 'q2', 'q3', 'q4'] as const
return KEYS.map(k => simpleAnswer(k))
```

### 3. Dérivation de clé

Si convention de nommage prédictible (`q1` → `a1`) :
```ts
const answer = k.replace('q', 'a')
```

### 4. Loop sur variantes

Reconnaître énumérations cachées :
```ts
['public', 'shared', 'private'].map(v => (
  <p><strong>{t(`a6_${v}_label`)}</strong> : {t(`a6_${v}`)}</p>
))
```

### 5. Template strings / HTML strings

Backticks répétés 3+ fois (email, OG image, PDF, CSV, prompts LLM, logs structurés) :
```ts
const renderField = (label: string, value: string) => `<p><strong>${label} :</strong> ${value}</p>`
const fields: ReadonlyArray<[string, string]> = [['Nom', name], ['Email', email]]
return fields.map(([l, v]) => renderField(l, v)).join('')
```

### Anti-patterns à NE PAS faire

- Composant React full-blown pour 1 ligne (helper suffit)
- Composant partagé utilisé sur 1 seul site "pour préparer le futur" (3 occurrences min avant extraction)
- Refacto silencieux sans remonter (cassent confiance + invisibilité)
- Reproduire pattern dupliqué pour "cohérence avec existant" (pérennise la dette)

### Signaux d'alerte

- 3+ lignes consécutives même shape syntaxique (JSX, template, `<View>` PDF, `array.push`)
- Objet/tableau littéral 5+ entrées même structure
- 4+ entrées d'objet littéral même shape

→ analyse duplication obligatoire avant continuer.

## State Management

| Choix | Quand |
|---|---|
| **Zustand** | Global client state (theme, sidebar open, current user UI prefs) |
| **TanStack Query** | Server state SPA/Vite (queries, mutations, infinite scroll, cache invalidation) |
| **RSC + fetch caching** | Next.js App Router server data — évalue d'abord avant TanStack Query |
| **React Context** | DI léger (theme provider, locale, auth user resolved) — pas pour data |
| **Jotai** | NE PAS utiliser sauf demande user explicite ou projet déjà en place |
| **`useState`/`useReducer`** | État local d'un composant ou view model |

**Ne pas** : Redux/Toolkit (legacy), MobX (legacy), `useReducer` global comme remplaçant Zustand.

**Décision Next.js TanStack Query** : ASK avant d'ajouter. RSC + `fetch` cache + `revalidatePath` est souvent suffisant.

## Data Fetching

- **TanStack Query** SPA/Vite : `useQuery`, `useMutation`, infinite scroll, cache.
- **RSC + streaming** Next.js App Router : Suspense boundaries avec fallbacks meaningful.
- **Suspense** partout possible : remplace `isLoading` flag, narrowing automatique du type post-Suspense.

Pattern type :
```tsx
const UserProfile = () => (
  <ErrorBoundary fallback={<ErrorFallback />}>
    <Suspense fallback={<Skeleton />}>
      <UserProfileContent />
    </Suspense>
  </ErrorBoundary>
)
```

## Forms

| Cas | Choix |
|---|---|
| Form simple, peu de feedback | Server Action + `useActionState`/`useFormStatus` (Next.js) |
| Form complexe, validation real-time, multi-step, optimistic | RHF + Zod (`zodResolver`) |
| Form dynamique avec champs conditionnels lourds | TanStack Form (alternative) |
| Validation custom non couverte par Zod | Zod custom refine, **JAMAIS** logique custom hors Zod |

**Schema-first** : Zod schema source de vérité, `type X = z.infer<typeof XSchema>`.

**Décision Next.js** : évaluer Server Actions D'ABORD. Si form = simple sans optimistic UI ni validation client lourde, Server Action + `useActionState` plus productif. RHF si validation real-time UX importe, multi-step, conditional logic, ou optimistic updates requis.

Pas de default `useForm` en Next.js sans avoir évalué Server Action.

## Error Handling

**Stratégie par couche** (pas Sentry comme stratégie principale, Sentry = monitoring layer) :

| Couche | Stratégie |
|---|---|
| **Domain** | `Result<T, E>` ou unions `T \| null` — JAMAIS throw dans fonction pure |
| **Infrastructure** | Throw erreurs typées (seul endroit autorisé). `class ApiError extends Error { constructor(readonly status: number, message: string) {...} }` |
| **Application (hooks)** | Laisser remonter erreurs infra vers React via TanStack Query (`error` field) ou RSC. NE PAS avaler |
| **UI** | ErrorBoundary capture+affiche, Suspense pour chargement |

**Stack lib** : `react-error-boundary` pour ErrorBoundary fonctionnel.

**Pattern type** :
```tsx
<ErrorBoundary fallback={<ErrorFallback />}>
  <Suspense fallback={<Skeleton />}>
    <UserProfileContent />
  </Suspense>
</ErrorBoundary>

const UserProfileContent = () => {
  const user = use(fetchUser()) // React 19 — throw la Promise, throw l'erreur
  return <div>{user.email}</div>
}
```

**Anti-pattern** : `try/catch` dispersés dans composants, `setState({error})` géré manuellement, `console.error` swallowing.

## Testing

| Outil | Usage |
|---|---|
| **Vitest** | Default, toujours. Plus rapide que Jest, ESM-first. |
| **Jest** | Fallback uniquement si Vitest impossible (legacy monorepo) — ASK user avant |
| **MSW** | Mock API HTTP. ASK user si déjà setup ; sinon ASK install ou mocks classiques |
| **Testing Library** | `@testing-library/react` pour rendering + queries |
| **Playwright** | E2E navigateur |

**Helpers de test dupliqués** = priorité refacto identique au code prod (render avec providers, fixture builders bougent ensemble).

## Styling

- **Tailwind** + design token layer (CSS variables pour colors, spacing, radii)
- **CSS Modules** fallback pour animations complexes ou scoped overrides
- **No class string duplication** : Button = composant, pas `className="px-4 py-2 bg-blue-500"` copy-paste partout
- **CSS-in-JS lib** (styled-components, emotion) : NE PAS utiliser en nouveau projet (legacy uniquement)

Token layer pattern :
```css
:root {
  --color-primary: oklch(...);
  --radius-md: 0.5rem;
  --space-4: 1rem;
}
```
Tailwind config :
```ts
colors: { primary: 'var(--color-primary)' }
```

## Accessibility

- **ARIA patterns via Radix primitives** — ne jamais roll your own pour Select/Combobox/Dialog/Popover
- **Keyboard nav + focus management** sur modals, drawers, toasts
- **Headless UI** alternative à Radix
- **WCAG 2.1 AA** minimum sur projets nouveaux

## Performance

- **React 19 first** : `use` hook + React Compiler. Pas de `useMemo`/`useCallback` préemptif.
- `useMemo`/`useCallback` SEULEMENT si profilage prouve un perf issue.
- **Code splitting** : `dynamic` imports pour routes + heavy components (charts, editors).
- **Virtualisation** : TanStack Virtual pour long lists (>100 items).
- **Measure before optimize** — pas de premature work.

Pour règles perf détaillées (Promise.all, RSC waterfalls, bundle, hydration), voir skill `vercel-react-best-practices`.

## Component Patterns

- **Compound components** pour UI complexe (Select, Tabs, Modal, Accordion)
  ```tsx
  <Tabs.Root>
    <Tabs.List>
      <Tabs.Trigger value="a">A</Tabs.Trigger>
    </Tabs.List>
    <Tabs.Content value="a">...</Tabs.Content>
  </Tabs.Root>
  ```
- **Headless UI / Radix** pour primitives accessibles (Dialog, Popover, DropdownMenu, Tooltip)
- **Composition > inheritance** toujours
- **Forwarded refs** pour wrapping primitives HTML : `forwardRef<HTMLButtonElement, Props>(...)`
- **`ComponentPropsWithoutRef`** pour étendre props HTML :
  ```ts
  type ButtonProps = ComponentPropsWithoutRef<'button'> & { variant: 'primary' | 'ghost' }
  ```

## Feature-First DDD Layering

Frontend lives inside the same vertical slice as the backend. Dependencies point INWARD; outer imports inner, never the reverse.

```
domain/                  -> nothing. Business types, rules, value objects.
use-cases/               -> domain/ + own infrastructure/repository. Hooks here hold business logic (camelCase, e.g. useCharacterChat).
infrastructure/          -> domain/ + shared/infrastructure. Adapters, repositories.
presentation/components/ -> use-cases/ + infrastructure/. Cross-feature OK here ONLY.
presentation/actions/    -> 'use server'. Thin wrappers: auth + call use-case + error mapping. NEVER business logic.
presentation/view-models/-> UI-shape mapping for components. No I/O.
```

**Iron rules:**
- A `use-case` hook never imports `infrastructure/` of another feature, and a component never reaches into another feature's `use-cases`/`infrastructure`. Cross-feature happens at `presentation/` or via an action/API.
- **Server actions are thin.** `'use server'` file = auth check + call the use-case + map errors. Any branching/business logic belongs in `use-cases/`.
- **Business hooks vs view-models:** logic + data fetching → `use-cases/hooks/`. Pure UI-shape derivation → `presentation/view-models/`. Don't mix.
- Port/dep types are owned by the use-case, not by the infra adapter it calls.
- A `domain/` file imports nothing external (no React, no fetch, no Zod-from-schema).

Companion backend rule set: `pp-backend-developer` → DDD Layering. Same arrows, same smells.

## TypeScript Boundaries (frontend-specific)

Pour règles TS générales, voir `mastering-typescript`. Spécificités frontend :

- **Branded types** pour IDs métier : `type UserId = string & { readonly _brand: 'UserId' }`
- **`useRef<HTMLDivElement>(null)`** quand DOM pas encore monté
- **Context sans undefined hack** :
  ```ts
  const ctx = createContext<UserContextType | null>(null)
  const useUser = () => {
    const v = useContext(ctx)
    if (!v) throw new Error('useUser must be used within UserProvider')
    return v
  }
  ```
- **Event handlers typés** : `type InputProps = { onChange: (value: string) => void }` plutôt que `ChangeEvent` partout

### Monorepo

- Types partagés dans `packages/types` ou `packages/shared`, jamais copiés
- Drizzle/Prisma source de vérité pour types DB
- `openapi-typescript` pour contrats REST
- **tRPC** quand backend TS (zéro codegen, types end-to-end)
- `graphql-codegen` si GraphQL déjà en place

## Checkpoints

Pause et confirme avant :
- **Refacto duplication** touchant >3 fichiers — montrer liste fichiers
- **Switch lib state mgmt** (Zustand → autre, ou ajout TanStack Query en Next.js)
- **Bulk replace** `class` → function component sur projet legacy
- **Migration RHF ↔ Server Actions** sur formulaires existants
- **Adding ErrorBoundary global** — affecte toute l'app
- **Switch Vitest ↔ Jest** ou install MSW
- **Bulk migration** Redux → Zustand, styled-components → Tailwind (touche tous composants)
- **Adding Radix/Headless** sur composants existants custom (changement API publique)
- **Désactiver React Compiler** ou downgrade React 19 → 18 (impact perf général)
- **Convert un dossier `presentation/` complet en RSC** sans isoler les islands client

Single-file fix : apply directly, cite pattern.

## Output Format

Diff : `// {pattern}: {one-line why}`
Review : `path:line — {pattern}: {problem}. {fix}.`

**Recall vs precision** : flag speculative matches (possible duplication, possible scope leak) avec `[speculative]`.

## Worked Example

**Input:** "Add toast notification when user saves settings."

1. *Classify* → "extract repeated JSX / inline style / helper" + duplication detect (Regle 0ter from agent).
2. *Scan trigger* → `const showToast = ...` declared inline ? Grep repo first.
3. *Quantify*: `grep -r 'showToast' src/` → 3 redeclarations + shared `toast.success()` already exists in `components/ui/toast`.
4. *STOP — duplication detected*:
   ```
   Pattern : inline showToast helper
   Occurrences : 3 fichiers (settings, profile, billing)
   Existing : components/ui/toast.tsx exports toast.success/error/info
   Proposition : migrer 3 sites vers shared, supprimer helpers inline
   Action : (A) refacto / (B) note dette / (C) leave
   ```
5. Wait user. If A: replace `showToast(msg)` → `toast.success(msg)`, run LSP `findReferences` to verify all sites covered.
6. *Cite* `// duplication-catalog: shared toast.success replaces 3 inline helpers`.

## Fallbacks & Edge Cases

- **Project uses different stack** (Redux, MobX, styled-components) → respect existing convention, don't bulk-rewrite. Flag once, defer to user.
- **Lib not in stack opinions** → ASK user avant ajouter. Pas de default silencieux.
- **TS rule conflict** → defer to `mastering-typescript` skill.
- **Next.js perf rule conflict** → defer to `next-best-practices`.
- **React perf rule conflict** → defer to `vercel-react-best-practices`.
- **Pure clean architecture question** → defer to `clean-ddd-hexagonal`.
- **Test in `*.spec.ts`** → relax some prod rules (mocks OK, MSW handlers fine even if "duplicated" across tests).
