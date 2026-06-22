# Documentation Guidelines — zava-storefront

What to document, where, and how much. The bar is: "can a teammate
understand this without asking you?"

---

## 1. Every exported lib/ function gets a JSDoc comment

One mandatory line: what the function does. Add `@param` for any
parameter whose purpose isn't obvious from the name. Add `@throws` when
the function throws on a specific condition (e.g. quantity limit,
zero total).

```ts
/**
 * Applies a discount code to a subtotal and returns the discount amount in cents.
 * Returns 0 for unknown codes.
 * @param subtotalCents - Cart subtotal before discount, in integer cents.
 * @param code - Discount code string, or null for no discount.
 */
export function applyDiscount(subtotalCents: number, code: string | null): number
```

Keep it short. One sentence is fine. Do not restate what the type
signature already says.

---

## 2. New API routes get a header comment

Three lines at the top of every new `route.ts`:

```ts
// GET /api/products?limit=&offset=
// Returns a paginated list of non-archived products.
// Auth: public (no authentication required)
```

If the route requires authentication, say so explicitly. "Auth: requires
session cookie" or "Auth: public" — one of those two, no ambiguity.

---

## 3. Non-obvious business rules get an inline comment

Tax rates, discount thresholds, quantity caps, money rounding —
if the number or condition isn't derivable from the variable name,
add a comment. Future readers should not have to google why `0.25`
is the VIP discount:

```ts
if (upper === 'VIP25' && subtotalCents >= 10_000) // 25% off orders over €100
```

The threshold for "non-obvious" is: would you be able to explain this
to a new teammate in ten seconds? If no, add the comment.

---

## 4. New environment variables are documented in the README

Any new `process.env.FOO` must appear in the **Environment Variables**
section of the README before (or in the same PR as) the code that reads it.
Include: the name, what it controls, whether it's required or optional,
and an example value (never a real value).

---

## 5. User-visible changes update the README

If the work item changes or adds a user-visible behaviour — a new API
endpoint, a new discount type, a changed response shape — the README
gets a corresponding update in the same PR. Documentation lag is tech debt.

---

## 6. Zod schemas are documentation — keep them close to their types

The schema IS the contract for what the function accepts. Keep
`fooSchema` and `type Foo = z.infer<typeof fooSchema>` in the same file,
adjacent to each other. Reviewers should be able to read the schema
and immediately understand what the API or function accepts.

---

## 7. Test names describe the expected behaviour, not the inputs

```ts
// good — reads as a spec
it('returns 0 when discount code is null')
it('caps quantity at 99 and throws above')

// weaker — just restating the call
it('applyDiscount with null code')
it('addItem with quantity 100')
```

Test names are documentation. When a test fails in CI, the name is
the first thing the reader sees.
