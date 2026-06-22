# Architecture Guidelines — zava-storefront

How this codebase is layered and why. Follow these rules when adding
or changing code so the structure stays coherent.

---

## Layer map

```
app/api/**/route.ts    — HTTP boundary: validate input, call lib/, return JSON
app/**/*.tsx           — Server components: fetch from lib/db directly
lib/*.ts               — Business logic: pure TS, no Next.js imports
tests/*.test.ts        — Unit tests for lib/ functions
```

---

## 1. lib/ is a Next.js-free zone

Files under `lib/` must not import from `next/server`, `next/navigation`,
or any other `next/*` package. They import TypeScript, `zod`, `pg`, and
each other. This is what makes them unit-testable in Vitest without
spinning up a Next.js process.

If you find yourself reaching for a Next.js type in lib/, the logic
belongs in the route handler instead.

---

## 2. Route handlers are thin

A route handler in `app/api/` does exactly three things:

1. Parse and validate the request with Zod (`safeParse`, 400 on failure).
2. Call one (or a small number of) `lib/` function(s).
3. Return a `NextResponse.json(...)`.

No business logic, no SQL, no conditional branches based on domain rules.
If the handler body is growing, the logic belongs in a new lib/ function.

---

## 3. Server components call lib/db directly — not their own API routes

`app/page.tsx` and other server components import `{ db }` from
`@/lib/db` and call `lib/` functions directly. They do not fetch
`/api/products` over HTTP. An HTTP round-trip from server component to
its own route handler is wasted latency and unnecessary serialization.

---

## 4. Inject db into lib/ functions — do not import it at call sites

Functions that need the database take `db: Db` as their first parameter:

```ts
// correct — db injected, testable
export async function createOrder(db: Db, input: OrderInput): Promise<Order>

// wrong — hard-coded import, impossible to test without a real DB
import { db } from './db';
export async function createOrder(input: OrderInput): Promise<Order>
```

The only place that imports `{ db }` from `lib/db` is the route handler
or server component that starts the call. Tests pass a mock or in-memory db.

---

## 5. Zod schemas live in lib/ next to their types

Define schemas in the same `lib/` file as the types they describe —
not inside route handlers. Route handlers import the schema:

```ts
// in lib/orders.ts
export const orderSchema = z.object({ ... });
export type OrderInput = z.infer<typeof orderSchema>;

// in app/api/orders/route.ts
import { orderSchema } from '@/lib/orders';
const parsed = orderSchema.safeParse(await req.json());
```

---

## 6. New external dependencies need a justification comment in package.json

Before `npm install <package>`, answer in one sentence: what does it do,
and why can't we use what's already there? Write that sentence as a comment
in `package.json` next to the dependency (or in the PR description if
`package.json` doesn't support comments). Don't add a dependency for
something `zod`, native `crypto`, or stdlib already handles.

---

## 7. Tests mirror the lib/ structure

For every `lib/foo.ts` there is (or should be) `tests/foo.test.ts`.
Tests import only from `lib/`; they never import from `app/`. They inject
a mock `db` — they do not connect to a real database.
