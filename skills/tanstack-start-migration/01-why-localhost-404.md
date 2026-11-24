# 01 — Why localhost:5173 returned 404

## Symptom
`pnpm dev` starts, but every URL on `http://localhost:5173` returns **404**.

## Cause
The TanStack Start plugin was never loaded.

- `vite.config.ts` only contained Vite+ lint/format settings — **no `plugins`**.
- The real TanStack config (`tanstackStart()`, `viteReact()`) had been pasted into
  **`next.config.ts`**, a file Vite never reads.

Without the plugin, `vite dev` is a plain static server: no `index.html`, no SSR
handler, no route tree → 404 for everything.

## Fix
Put the plugins in `vite.config.ts` (keep the Vite+ settings beside them):

```ts
import { defineConfig } from 'vite-plus';
import { tanstackStart } from '@tanstack/react-start/plugin/vite';
import viteReact from '@vitejs/plugin-react';
import tailwindcss from '@tailwindcss/vite';
import tsconfigPaths from 'vite-tsconfig-paths';

export default defineConfig({
  server: { port: 5173 },
  plugins: [
    tsconfigPaths(),   // makes `@/...` imports work (tsconfig "paths")
    tailwindcss(),
    tanstackStart(),   // must come BEFORE viteReact()
    viteReact(),
  ],
  // staged / fmt / lint: keep as before
});
```

Then delete `next.config.ts`.

## How to tell it's fixed
`src/routeTree.gen.ts` appears automatically. That file is generated **only** by the
TanStack plugin — if it exists, the plugin is running.

## Other things found on the same branch
- `package.json` had **two `"scripts"` blocks** — JSON keeps only the last one, so
  `"prepare": "vp config"` was silently lost. Merge them.
- Two lockfiles (`package-lock.json` + `pnpm-lock.yaml`) — keep one.
- Route files were moved from `app/` with **zero changes**. TanStack does not understand
  `page.tsx` / `layout.tsx`; it wants `index.tsx` / `route.tsx` and every file must
  `export const Route = createFileRoute(...)`.

## Lesson
When a dev server returns 404 for *everything*, the framework isn't wired up at all —
check the bundler config before looking at routes.
