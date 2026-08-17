# eliminating-ai-slop

A [Claude Skill](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
that runs a disciplined clarify → implement → verify workflow on coding
tasks: it asks targeted questions only when requirements are genuinely
ambiguous, keeps diffs surgical and matched to the existing codebase's
style, then detects and runs the project's real test/lint command and
self-reviews the diff before calling anything done.

It exists to cut down on "AI slop" — unrequested abstractions, unasked-for
config knobs, drive-by reformatting, and confidently-claimed-but-never-run
verification — without turning every one-line fix into an interview.

## Install

**Claude Code (per-project or personal skills directory):**

```bash
git clone https://github.com/damiankluk/eliminating-ai-slop.git \
  ~/.claude/skills/eliminating-ai-slop
```

Or, scoped to a single project:

```bash
git clone https://github.com/damiankluk/eliminating-ai-slop.git \
  .claude/skills/eliminating-ai-slop
```

Claude discovers the skill automatically from `SKILL.md`'s frontmatter —
no further configuration needed. It activates on coding tasks where scope
is ambiguous, or when you explicitly ask for a surgical/no-slop diff.

**Claude apps / API:** upload this directory as a Skill per the
[Skills quickstart](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/quickstart).

## Structure

```
SKILL.md                          — the skill itself (frontmatter + workflow)
references/verification-commands.md — per-ecosystem test/lint detection + fallback
references/examples.md            — worked examples (compliant vs. non-compliant diffs)
```

## License

MIT — see [LICENSE](LICENSE).
