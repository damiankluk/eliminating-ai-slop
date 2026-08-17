# Worked examples

## Contents
- When to ask vs. when to skip the interview
- Compliant vs. non-compliant diff
- The After-Action loop in practice

## When to ask vs. when to skip the interview

**Task: "Fix the typo on line 42, it says 'recieve'."**

Skip Stage 1 entirely: unambiguous, single-line, no design decision.
Just fix it, run the verification command, done.

**Task: "Add rate limiting to the API."**

This goes through Stage 1. A reasonable clarifying message, all questions
batched together:

> Before I implement this, three things affect the design:
> 1. Per-user, per-IP, or per-API-key? (I don't see an existing identity
>    concept for this in the codebase.)
> 2. Two paths for where the limit state lives:
>    - **Minimal:** an in-process counter. A few lines, resets on
>      restart, fine as long as this runs as a single instance.
>    - **Extensible:** a Redis-backed counter. Survives restarts and
>      works across instances, which may matter since
>      `docker-compose.yml` already runs this behind a load balancer.
>    Which one?
> 3. What should happen when the limit is hit: HTTP 429 with a
>    `Retry-After` header, or something else your clients already expect?

Not asked: "What language/framework should I use?" That's answerable by
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

## The After-Action loop in practice

**Task: "Add the Redis-backed limiter from the example above."**

First run of the verification command fails. Instead of guessing again,
work the loop:

> **What happened?** `pytest` fails:
> `redis.exceptions.ConnectionError: Error 111 connecting to
> localhost:6379. Connection refused.`
>
> **Why?** The test suite assumes a Redis instance is already running.
> `docker-compose.yml` defines one, but it isn't up in this shell.
>
> **Corrective action:** started it with
> `docker compose up -d redis`, then reran `pytest`. Now failing on a
> second, different error.

That second failure gets its own turn through the same three questions.
It doesn't get patched with the same fix that just failed, and it
doesn't get skipped: the loop only ends when the command passes or when
the cause turns out to need a decision only the user can make (in which
case that becomes a fresh, targeted question, not a guess).
