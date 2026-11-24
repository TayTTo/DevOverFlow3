# 02 — Reading a Vite error and reasoning to the fix

## The error
```
SyntaxError: [vite] Named export 'Space_Grotesk' not found. The requested module
'next/font/google' is a CommonJS module, which may not support all module.exports
as named exports.
    at ...module-runner.js...
    at async eval (src/routes/__root.tsx:2:1)
    at async eval (src/routeTree.gen.ts:11:1)
```

## Step 1 — Read the stack trace bottom-up
The **bottom** is what Vite was doing; the **top** is where it broke.

| Line | What it tells you |
|---|---|
| `routeTree.gen.ts:11` | The route tree exists → the TanStack plugin works (lesson 01 fixed) |
| `module-runner.js` | Vite's SSR runner → TanStack Start is rendering on the server |
| `__root.tsx:2:1` | The exact failing line: `import { Inter, Space_Grotesk } from "next/font/google"` |

So: routing is fine, one import is the problem. A new error is often **progress**.

## Step 2 — Understand the message
`next/font/google` *does* resolve (Next is still in `node_modules` because `next-auth`
depends on it), but it is an **empty placeholder**. Next's compiler (SWC) rewrites
`next/font` calls at build time. No Next compiler → no rewrite → no export.

**General rule:** anything Next.js does *at compile time* breaks under Vite:
`next/font`, `next/image` optimization, `export const metadata`, `"use client"`,
async server components.

## Step 3 — Don't trust the suggested fix
Vite suggests:
```ts
import pkg from 'next/font/google';
const { Inter, Space_Grotesk } = pkg;
```
That only silences the syntax error — `Space_Grotesk` would be `undefined` and crash
on the next line. Vite's hint is generic CommonJS advice; it doesn't know what the
package is for.

> When an error's fix is about *how* to import something, first ask whether you
> *should* be importing it at all.

## Step 4 — Replace what the thing did
`next/font` did two jobs: load font files + set a CSS variable. Do both yourself
→ see [03 — Fonts from npm](03-fonts-from-npm.md).

## Step 5 — Find the rest in one go
Vite stops at the first broken import. List them all instead of one per reload:
```bash
git grep -n "from ['\"]next" -- src components
```

Next → TanStack replacements:

| Next | TanStack / plain React |
|---|---|
| `next/link` `<Link href>` | `@tanstack/react-router` `<Link to>` |
| `next/image` `<Image>` | `<img>` |
| `useRouter().push(x)` | `useNavigate()` → `navigate({ to: x })` |
| `usePathname()` | `useLocation().pathname` |
| `useSearchParams()` | `Route.useSearch()` |
| `next/dynamic` | `React.lazy` + `<Suspense>` |
| `export const metadata` | `head: () => ({ meta: [...] })` on the route |
| `"use client"` | delete |

## Official reference
TanStack's Next migration guide (a checklist, no error walkthroughs):
https://tanstack.com/start/latest/docs/framework/react/migrate-from-next-js
Runnable example: TanStack/router repo → `examples/react/start-next-migration`.
