# Security Guidelines — zava-storefront

These are the checks that matter most for this codebase.
Both the work-item-builder and panel-review skills read this file.

---

## 1. Validate all request input at the route boundary with Zod

Every `app/api/*/route.ts` handler must parse incoming data through a
Zod schema **before** calling any `lib/` function. Use `safeParse`, never
`parse` directly — the handler should return a 400 before touching
business logic if the input is invalid.

```ts
// correct — validate first, then call lib/
const parsed = SomeSchema.safeParse(Object.fromEntries(req.nextUrl.searchParams));
if (!parsed.success) return NextResponse.json({ error: 'invalid_request' }, { status: 400 });
const result = await doTheThing(db, parsed.data);
```

Rejection also applies to URL path segments, headers, and body JSON.

---

## 2. All SQL via parameterized queries — no string interpolation

Use `db.query(sql, params)` with `$1 $2 …` placeholders every time.
Never build a query string by interpolating user input, even partially.

```ts
// correct
db.query('SELECT * FROM products WHERE id = $1', [id])

// wrong — never do this
db.query(`SELECT * FROM products WHERE id = '${id}'`)
```

This applies to dynamic `WHERE` clauses, `IN` lists, and `LIKE` patterns.
For `ILIKE` with wildcards, pass the `%` inside the param value, not in
the SQL string (see `lib/search.ts` for the right pattern).

---

## 3. No secrets in source code

All credentials go through `process.env.*`. New environment variables must
use names that are obviously non-secret in their name (e.g. `DATABASE_URL`,
not `DB_PASS_PROD`). No hardcoded tokens, API keys, or connection strings
anywhere in the codebase, including test fixtures.

---

## 4. Money is always integer cents

`priceCents`, `totalCents`, `discountCents`, `taxCents` — amounts are stored
and computed as integers. Never introduce a `price` field that holds euros or
dollars as a float, and never convert to float for a calculation.

If the work item asks for a new money field, model it as `*Cents: number`.

---

## 5. Route handlers that mutate data must verify the caller

Any handler that creates, updates, or deletes a record must check that the
request carries valid authentication before calling lib/. If no auth
middleware exists yet for the route, that is itself a blocking issue —
note it in the PR rather than shipping an unprotected mutation endpoint.

---

## 6. Generic error messages to clients, details to logs only

Route handlers return `{ error: 'short_snake_case_code' }` to the client.
The full error (message, stack, query params) goes to the server log.
No stack traces, no database error text, no field names from failed
Zod parses in the response body.

---

## 7. No PII or financial data in log output

Do not log `userId`, full email, card details, or order totals as
plain strings. If you need to trace a user action, log an opaque
correlation ID. Prices and totals stay out of logs.
