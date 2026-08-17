---
name: eliminating-ai-slop
description: Runs a disciplined pre-implementation, implementation, and verification workflow for non-trivial coding tasks. Asks targeted clarifying questions only when requirements are genuinely ambiguous, makes minimal surgical diffs that match the existing codebase's style, then detects and runs the project's actual test/lint commands and self-reviews the diff before finishing. Use when implementing a feature, fixing a bug, or refactoring code, especially when the user says "no AI slop", "keep the diff surgical", "match the codebase style", or when scope, requirements, or existing module behavior is unclear before writing code.
---

# Eliminating AI Slop

A three-stage discipline for coding tasks: clarify only what's genuinely
unclear, change only what the task requires, then prove the result works
instead of asserting it. The goal is a diff a senior reviewer would approve
without follow-up questions.

## Stage 0: Decide if this even applies

Skip straight to Stage 2 (no interview, no ceremony) when any of these hold:

- The task is small and localized (roughly under ~30 lines, one clear file
  or module, no new public API or dependency).
- Requirements are already unambiguous: the user specified what and where.
- The session is non-interactive/headless, a background/scheduled run, or a
  delegated subagent task with no one available to answer questions. State
  your assumptions in the response instead of asking.
- The user's own instructions (CLAUDE.md, this conversation) already grant
  broad autonomy. That takes precedence over Stage 1.
- You're already in an approved plan (plan mode, or a plan the user just
  confirmed). Don't re-ask what the plan already settled.

Everything else goes through Stage 1 first.

## Stage 1: Clarify before you build

Ask only when at least one is true, and say *why* you're asking:

- The task touches a module/behavior you can't confirm by reading the
  code. Not "I feel unsure," but a concrete gap: an undocumented function,
  an external contract, a data shape you'd have to guess.
- Two or more implementations are genuinely viable with different real
  tradeoffs (data model shape, public API surface, sync vs. async, new
  dependency vs. hand-rolled), not just style preference.
- The requested outcome is underspecified (e.g., "handle errors better"
  with no definition of "better").

Ask at most 3-4 questions, batched in one message, not asked serially.
When two implementation paths are both reasonable, name both briefly and
ask which one, rather than picking silently. Don't interview over things
answerable by reading the repo: check first, ask second.

## Stage 2: Surgical implementation

- **Touch only what the task requires.** No reformatting, renaming, or
  "while I'm here" cleanup of code the task didn't ask about.
- **Dead code you created is your responsibility.** If your change orphans
  an import or function *in a file you already touched*, remove it. Leave
  unrelated dead code in untouched files alone. Flag it in your summary
  instead of deleting it there.
- **No unrequested flexibility.** No config options, abstractions, or
  "for the future" generality nobody asked for. A one-off stays inline.
- **Match the codebase, not a style guide.** Read the surrounding file(s)
  first for naming convention, error-handling pattern, and module layout,
  then mirror them exactly, even if you'd choose differently on a blank
  page.
- **New dependencies:** propose the dependency and the one-line reason in
  your response. In an interactive session, wait for a yes on anything
  non-trivial. In a non-interactive/headless run, proceed only if it's
  the obvious idiomatic choice for this stack (e.g. adding `pytest` to a
  Python test suite that has none) and say what you added and why.

## Stage 3: Verify, don't assert

1. **Find the real verification command.** Don't guess. Detect it from
   the repo's own config (see
   [references/verification-commands.md](references/verification-commands.md)
   for the detection table across ecosystems and the fallback when no
   test/lint setup exists at all).
2. **Run it.** If it fails, loop: read the actual error/stack trace, form
   one concrete hypothesis for the cause, make the smallest fix that
   addresses it, rerun. Repeat until it passes or you're genuinely stuck.
   Don't repeat the same fix twice.
3. **Self-review the diff before handing it back.** Read your own
   `git diff` and remove: leftover debug/trace output in whatever form the
   language uses (`console.log`, `print`, `fmt.Println`, `dd()`,
   `debugger`, commented-out code you left as a note to yourself),
   comments that just restate the code, and anything not required by the
   task.

See [references/examples.md](references/examples.md) for a compliant vs.
non-compliant diff and a sample clarification exchange.

## Done means checkable, not vibes

Before reporting the task complete, confirm (and say so) that:

- The detected test/lint/build command was actually run and its exit
  code was 0 (or: no such command exists in this repo, and you say that
  plainly instead of implying verification happened).
- `git diff --stat` touches only files relevant to the stated task, no
  incidental files.
- Every new external dependency in the diff was named and justified in
  your response; none snuck in unmentioned.
