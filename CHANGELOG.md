# Changelog

## 1.1.0 (2026-08-17)

Restored two techniques from the original draft that the 1.0.0 rewrite
had generalized into vaguer language while fixing the red-team findings:

- Stage 1 now names the two-path framing explicitly again: for a real
  design fork, propose a minimal path and an extensible path, each with
  a one-line tradeoff, instead of a soft "name both if reasonable."
  Still scoped to the objective trigger (a genuine tradeoff), so it
  doesn't reintroduce a blocking gate on trivial tasks.
- Stage 3's fix loop is now an explicit three-step After-Action loop
  (What happened? Why? Corrective action?) instead of one compressed
  sentence, matching the original's checklist-style rigor.

## 1.0.0 (2026-08-17)

Initial public release. Rewritten in English from an internal Polish draft
("Anti-Slop & Deterministic Code Engineering v2.0.0") after a red-team pass
that surfaced 13 issues. Key changes from the draft:

- Added an explicit skip path (Stage 0) for small/unambiguous tasks and for
  non-interactive/headless/subagent runs. The draft's mandatory interview
  had no exit and would block automation.
- Replaced subjective triggers ("100% confidence", "shadow of doubt") with
  concrete, checkable conditions for when to ask questions.
- Added a precedence rule: explicit user instructions and active plan mode
  outrank the clarification stage instead of silently conflicting with it.
- Resolved the contradiction between "no drive-by refactoring" and
  "delete orphan code" by scoping dead-code removal to files already
  touched by the task.
- Replaced the hardcoded `npm`/`pytest`/`cargo` test commands with an
  ecosystem-detection table (references/verification-commands.md) covering
  ten+ ecosystems, plus an explicit fallback for repos with no test setup.
- Softened the absolute "no new dependency without consent" rule into a
  propose-and-justify rule, since the absolute form is unresolvable in
  non-interactive runs.
- Generalized debug-statement cleanup beyond `console.log`/`print` to any
  language's trace output.
- Removed named-individual attributions and undefined slang ("Matt Pocock",
  "Karpathy", "ponytail architecture", "GSD") in favor of self-explanatory
  section names.
- Tied success criteria to inspectable artifacts (verification command exit
  code, `git diff --stat` scope, explicit dependency list) instead of
  unverifiable claims like "clean" or "expected".
- Split into SKILL.md + references/ per Claude Skills authoring guidance,
  with worked examples (references/examples.md).
