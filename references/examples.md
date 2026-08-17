# Worked examples

## Contents
- When to ask vs. when to skip the interview
- Compliant vs. non-compliant diff

## When to ask vs. when to skip the interview

**Task: "Fix the typo on line 42, it says 'recieve'."**

Skip Stage 1 entirely — unambiguous, single-line, no design decision.
Just fix it, run the verification command, done.

**Task: "Add rate limiting to the API."**

This goes through Stage 1. A reasonable clarifying message, all questions
batched together:

> Before I implement this, three things affect the design:
> 1. Per-user, per-IP, or per-API-key? (I don't see an existing identity
>    concept for this in the codebase.)
> 2. Is there a preference between an in-process limiter (simple, resets
>    on restart, fine for a single instance) vs. a Redis-backed one
>    (survives restarts, works across multiple instances — you're running
>    behind a load balancer per `docker-compose.yml`, so this may matter)?
> 3. What should happen when the limit is hit — HTTP 429 with a
>    `Retry-After` header, or something else your clients already expect?

Not asked: "What language/framework should I use?" — that's answerable by
reading the repo, so it's checked, not asked.

## Compliant vs. non-compliant diff

**Task: "The `formatPrice` function crashes on `null` input."**

Non-compliant (AI slop): rewrites the whole file, introduces a generic
`safeFormat` abstraction nothing else calls, reformats unrelated
functions, adds a config object for future currencies nobody requested.

```diff
- function formatPrice(amount, currency) {
-   return currency + amount.toFixed(2);
- }
+ // Generic safe formatting utility for future extensibility
+ function safeFormat(value, formatter, fallback = '') {
+   if (value === null || value === undefined) return fallback;
+   try {
+     return formatter(value);
+   } catch (e) {
+     console.log('formatting failed', e);
+     return fallback;
+   }
+ }
+
+ const CURRENCY_CONFIG = { default: 'USD', supported: ['USD', 'EUR', 'GBP'] };
+
+ function formatPrice(amount, currency = CURRENCY_CONFIG.default) {
+   return safeFormat(amount, (v) => currency + v.toFixed(2), 'N/A');
+ }
```

Compliant: the smallest change that fixes the actual crash, matching the
existing style.

```diff
  function formatPrice(amount, currency) {
+   if (amount == null) return '';
    return currency + amount.toFixed(2);
  }
```

The second version is what Stage 2 and the self-review in Stage 3 are
for: no new abstraction, no unrequested config, no debug `console.log`,
no unrelated reformatting.
