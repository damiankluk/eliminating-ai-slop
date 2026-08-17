# Verification command detection

## Contents
- Detection table by ecosystem
- Fallback when no test/lint setup is found
- Notes on monorepos and multiple ecosystems

## Detection table by ecosystem

Detect the ecosystem from the files actually present in the repo (or the
nearest package root above the changed files), then prefer a script the
project already defines over a raw tool invocation.

| Signal file(s) | Ecosystem | Prefer (if defined) | Otherwise run |
|---|---|---|---|
| `package.json` | Node/JS/TS | `npm run test` / `npm run lint` (check `scripts` first; also try `pnpm`/`yarn` if that lockfile is present) | `npx vitest run`, `npx jest`, or `npx eslint .` depending on what's configured |
| `pyproject.toml`, `setup.py`, `requirements*.txt` | Python | `tox`, or a `Makefile`/`justfile` target | `pytest`, `python -m pytest`, `ruff check .`, `mypy .` — whichever configs exist (`pytest.ini`, `ruff.toml`, `mypy.ini`) |
| `Cargo.toml` | Rust | — | `cargo test`, `cargo clippy` |
| `go.mod` | Go | — | `go test ./...`, `go vet ./...` |
| `pom.xml` | Java (Maven) | — | `mvn test` |
| `build.gradle`, `build.gradle.kts` | Java/Kotlin (Gradle) | `./gradlew` if present | `./gradlew test` or `gradle test` |
| `Gemfile` | Ruby | `bin/rails test` if a Rails app | `bundle exec rspec`, `bundle exec rake test` |
| `*.csproj`, `*.sln` | .NET | — | `dotnet test` |
| `composer.json` | PHP | Check `composer.json` `scripts.test` | `composer test`, `vendor/bin/phpunit` |
| `mix.exs` | Elixir | — | `mix test`, `mix credo` |
| `CMakeLists.txt`, `Makefile` (C/C++) | C/C++ | `make test` if the target exists | project's documented build+test steps |

If a CI config exists (`.github/workflows/*.yml`, `.gitlab-ci.yml`,
`.circleci/config.yml`), it's the most reliable source of truth for the
real test/lint invocation — read it before guessing.

## Fallback when no test/lint setup is found

Do not fabricate a command and do not claim "tests passed" when none ran.
Instead:

1. Say explicitly that this repo has no automated test/lint/build
   configuration you could find.
2. Describe what you did to verify manually instead (e.g., traced the
   changed code path by hand, ran the script/binary once with sample
   input, checked syntax with the language's own `--check`/compile step
   if one exists).
3. If the change is non-trivial and genuinely warrants a test suite, say
   so as a observation — don't silently add a whole test framework as an
   unrequested dependency (see Stage 2's new-dependency rule in
   `SKILL.md`).

## Notes on monorepos and multiple ecosystems

Run the check scoped to the package/module you actually changed, not the
whole monorepo, unless the project's own tooling only exposes a
repo-wide command. If a monorepo has multiple ecosystems (e.g. a Python
backend and a JS frontend), only run the verification for the side you
touched.
