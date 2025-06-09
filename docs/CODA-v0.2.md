# CODA 0.2 · Component-Oriented Design Architecture  
*A component methodology that fuses design-intent validation with modern CSS layering, Storybook-first workflows, Atomic folder naming, and W3C design-token governance.*

---

## What’s new in 0.2 

| Enhancement | Source Inspiration | Why it matters |
|-------------|--------------------|----------------|
| **CUBE CSS layers** (`composition → utility → block → exception`) | CUBE CSS | Unambiguous styling cascade inside every component; fewer selector collisions. |
| **Storybook-first CDD** | Component-Driven Development | Generates living docs, visual tests, and prop controls automatically for every CODA module. |
| **Atomic folder taxonomy** | Atomic Design 2025 | Fast mental mapping for large teams: `atom/`, `molecule/`, `organism/`, `template/`, `page/`. |
| **W3C JSON design-tokens** | W3C Design-Tokens draft | Single source of truth consumed by Tailwind, CSS custom props, Figma, and CI linters. |

---

## Updated Core Principles

1. **Unified Encapsulation** – interface · implementation · styling · design intent all in one file.  
2. **CUBE Layer Clarity** – every style rule explicitly tagged as *composition*, *utility*, *block*, or *exception*.  
3. **Intent & Constraint Preservation** – runtime (dev-only) validation + TypeScript types.  
4. **Atomic Discoverability** – folder names signal component granularity.  
5. **Token Single Source** – W3C JSON tokens flow into Tailwind *and* CSS variables.  
6. **Executable Docs** – Storybook stories auto-generated from CODA metadata.  
7. **Extension via Decorators** – variants layer on without touching the base module.

---

## CODA 0.2 Module Skeleton

```tsx
// molecule/FeatureCard/FeatureCard.tsx
// --------------------------------------------------
// 1. MODULE INTERFACE
export interface IFeatureCard { /* … */ }

// 2. DESIGN INTENT METADATA (exported for Storybook addon)
export const DESIGN_INTENT = { /* … */ };

// 3. DEFAULT CONTENT
const DEFAULT_CONTENT = { /* … */ };

// 4. CONCRETE IMPLEMENTATION
class FeatureCardImpl implements IFeatureCard { /* … */ }

// 5. CUBE-LAYER STYLES (co-located)
/** @cube Composition */
@layer composition { .flow { display:flex; flex-direction:column; gap:var(--space-md) } }
/** @cube Utility */
@layer utilities   { .u-text-reset { color:inherit; font:inherit } }
/** @cube Block */
@layer blocks      { .featureCard { @apply bg-white rounded-lg shadow-md p-6; } }
/** @cube Exception */
@layer exception   { .featureCard--bordered { @apply border-2 border-current; } }

// 6. SERVICE REGISTRY
const reg = new WeakMap<object, IFeatureCard>();
export const getFeatureCard = (key: object = {}, c = {}) =>
  reg.get(key) ?? reg.set(key, new FeatureCardImpl(c)).get(key)!;

// 7. REACT WRAPPER (for apps + Storybook)
export const FeatureCard: React.FC<Partial<FeatureContent>> = props =>
  getFeatureCard({}, props).render();

// 8. METADATA EXPORT
export const __CODA_METADATA = { designIntent: DESIGN_INTENT, defaultContent: DEFAULT_CONTENT };
```

---

## Atomic Folder Layout Example

```
src/components/
├─ atom/
│  └─ Icon/
├─ molecule/
│  └─ FeatureCard/
├─ organism/
│  └─ HeroSection/
├─ template/
│  └─ LandingPage/
└─ page/
   └─ MarketingHome/
```

*Each directory contains its own CODA module(s), Storybook story, and tests.*

---

## Design-Token Pipeline

1. Author tokens in **`tokens.json`** (W3C draft format).  
2. Run **Style Dictionary** to output:  
   - `tailwind.config.js` theme section.  
   - `:root { --color-primary-500: … }` CSS variables.  
3. `ThemeService` reads **only CSS variables** at runtime—no hard-coded values.

---

## Storybook Integration

| Step | Automation |
|------|------------|
| **Story generation** | `npm run gen:stories` scans `src/components/**\/*.tsx` and writes `*.stories.tsx` with default, variant, and constraint-violation examples. |
| **Design-intent docs** | Custom addon reads `__CODA_METADATA` and renders a table in Storybook Docs tab. |
| **Visual regression** | `npm run test:visual` (via `@storybook/test-runner`) uses Chromatic or Playwright to snapshot each story. |

---

## Migration Checklist

1. **Tokenise** global colors/spacing → `tokens.json`.  
2. **Wrap** existing components into new CODA skeletons, tagging CUBE layers.  
3. **Place** modules in appropriate Atomic folder.  
4. **Generate** Storybook stories (`gen:stories`).  
5. **Add** decorator modules for each variant instead of prop conditionals.  
6. **Enable** dev-only design-intent validation; strip in prod build.