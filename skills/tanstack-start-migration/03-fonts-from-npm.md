# 03 — Installing fonts from npm (replacing next/font)

## Why no single doc covers this
Each tool owns one step, so each doc covers only its step:

| Step | Owner | Docs |
|---|---|---|
| Ship font files + `@font-face` CSS | **Fontsource** (npm package) | https://fontsource.org/docs/getting-started/install |
| Bundle the imported CSS | **Vite** | Vite guide → Features → CSS |
| Font name → `font-*` utility | **Tailwind** | Theme / font-family docs |
| Render `className="font-..."` | **React** | — (React never sees fonts) |

`next/font` did all four at once, so Next users never saw the seams.
Per-font install page example: https://fontsource.org/fonts/space-grotesk/install

## How Fontsource works
Every Google font is an npm package containing the font files + a CSS file with
`@font-face` rules. `import` the package → Vite bundles the CSS → fonts are served
from your own app (self-hosted, like `next/font`).

| | Variable (recommended) | Static |
|---|---|---|
| Package | `@fontsource-variable/space-grotesk` | `@fontsource/space-grotesk` |
| Weights | one file = all weights | one import per weight |
| Import | `import '@fontsource-variable/space-grotesk'` | `import '@fontsource/space-grotesk/700.css'` |
| CSS name | `'Space Grotesk Variable'` | `'Space Grotesk'` |

⚠️ The CSS name must match the package type. Wrong name = font silently not applied.

## Old setup → new setup
| Old (`next/font`) | New |
|---|---|
| Inter, weights 100–900 | `@fontsource-variable/inter` (100–900) |
| Space Grotesk, 300/400/500/700 | `@fontsource-variable/space-grotesk` (300–700) |
| `subsets: ["latin"]` | automatic — subsets split by `unicode-range`, browser downloads only Latin |
| `inter.className` on `<body>` | `font-inter` class on `<body>` |
| `spaceGrotesk.variable` | variable defined in `@theme`, global, no class needed |
| `display: swap` | Fontsource default |

## Steps

### 1. Install
```bash
pnpm add @fontsource-variable/inter @fontsource-variable/space-grotesk
```

### 2. `src/routes/__root.tsx`
Remove the `next/font/google` import and the `Inter(...)` / `Space_Grotesk(...)` calls.
Add:
```tsx
import '@fontsource-variable/inter';
import '@fontsource-variable/space-grotesk';
```
Change body:
```tsx
<body className="font-inter antialiased">
```

### 3. `globals.css` — first `@theme` block
```css
/* before — self-referencing; only worked because next/font set the value on <body> */
--font-inter: var(--font-inter);
--font-space-grotesk: var(--font-space-grotesk);

/* after */
--font-inter: 'Inter Variable', sans-serif;
--font-space-grotesk: 'Space Grotesk Variable', sans-serif;
```
Existing `font-inter` / `font-space-grotesk` classes in components keep working.

## Verify
- The two test `<h1>`s in `__root.tsx` render in different fonts.
- DevTools → Network → Font: `inter-latin-wght-normal…woff2`, `space-grotesk-latin-wght-normal…woff2` from localhost.
- DevTools → Elements → `<body>` → Computed `font-family`: `Inter Variable`.

## Known difference
`next/font` also generated a size-adjusted fallback font to reduce layout shift.
Fontsource doesn't — expect a tiny text shift on first load.

## Lesson
A CSS variable that references itself (`--x: var(--x)`) is a cycle → invalid → silent
fallback. If something "just worked" in Next, check whether Next was injecting the
real value from somewhere you never saw.
