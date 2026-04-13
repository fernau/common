# Shared instructions

## Process

- **Accuracy over helpfulness — verify and push back, don't just comply.** A golem that executes the letter of a request without questioning its framing is worse than an assistant that says "the framing is wrong." Before acting, check the task's framing, assumptions, and scope; push back when they're off. Get-there-itis fires on failures, but a working-but-wrong-framed task never fails — question framings deliberately, not reactively.
- **Read adjacent context before concluding.** The specific file/command the user names may not be what should exist. Read callers, config, deployment files, alternatives — decide whether the named scope is the right scope before acting on it.
- **Before building, state the simplest version that could work.** If the plan has multiple steps or transformations, ask whether a subset would suffice.
- **When an approach fails, re-examine the premise before patching.** Sunk cost in the current approach makes patching feel cheaper than reconsidering — resist this (get-there-itis). Step back and ask what the actual constraint is.
- **When the user corrects an approach, analyze root cause before implementing the fix.** Categorize: was this a detail error (AI should catch) or a systems/judgment error (human should catch)? Identify what systemic change prevents recurrence.

## Verify facts, don't guess

Don't rely on training data, heuristics, or convention when facts are easily accessible. If the answer is one command, one file read, or one web search away, look it up before writing code.

The cost of checking is seconds; the cost of guessing wrong is multiple correction rounds.

## Single responsibility principle

- Each function does one thing: query, format, parse, or cache — never multiple
- Tools/handlers orchestrate by delegating to helpers; they contain no business logic beyond assembling the call
- Helpers do not call other helpers — keep the call graph flat
- When a function starts doing two things, extract the second into its own helper

## No duplication of logic

- Each calculation or business rule must exist in exactly one place
- Data stored or returned must reflect final state — consumers must not re-apply upstream logic
- Some duplication in tests is acceptable (e.g. re-deriving expected values)

## Size limits — extract when exceeded

- Module: 300 lines
- Function: 100 lines
- Nesting: max 2–3 levels deep

### Ratchet pattern

Existing oversize modules/functions should have limits that reflect their current size. These limits must only go **down** over time — refactor to reduce size, then lower the limit. Never raise a limit.

## No silent error drops

Errors must be observable. Never redirect stderr to `/dev/null`, return silently on failure, or report success when a step was skipped. A silent failure is worse than a loud one — it produces wrong state that's only discovered much later, with no trail to diagnose.

- **Log, don't swallow.** If a subprocess can fail, capture its stderr and write it somewhere durable (log file, not just pod stdout). `2>/dev/null` is a last resort, not a default.
- **Distinguish success from skip.** If an operation was skipped (lock contention, missing config, optional dependency unavailable), return or log a distinct status — not the same "ok" as genuine success.
- **Surface errors to the caller.** For interactive tools, include failure/skip status in the response. For batch jobs, write to a log file that outlives the process.
- **Fail-safe, not fail-silent.** It's fine to continue past a non-fatal error (resilience). It's not fine to hide that the error happened.

## Security

- **Path traversal** — resolve all file paths with `Path.resolve()` or `expanduser()` and validate they fall within expected roots. Never blindly join user-supplied path components
- **No shell injection** — never use `subprocess` with `shell=True` or interpolate strings into shell commands
- **No credentials in code** — secrets, passwords, API keys must never appear in source. Use environment variables or external config
- **Bounded reads** — when reading files, be aware of file sizes. Don't load unbounded data into memory without limits
- **Input validation** — validate parameters before acting. Helpers should raise `ValueError` on bad inputs; handler functions must catch all exceptions and return clear error strings to clients

## Code style

- Full type annotations everywhere; use Python 3.10+ built-in generics (`list[dict]`, `dict[str, int]` — not `List`, `Dict`)
- Private helpers prefixed with `_`, named `_verb_noun()` (e.g. `_run_query`, `_format_results`)
- Google-style docstrings (Args / Returns) on all public functions and classes
- Module-level docstring in every file
- Import order: stdlib → blank line → external → blank line → system/local packages
- Raise `ValueError` with descriptive messages for invalid parameters in helpers (fail fast on bad args internally)
- Prefer simplicity and clear architecture over backwards compatibility

## Honest metrics over passing metrics

Metrics must reflect reality. Never game a metric to make it pass — an honest failure is more valuable than a hollow success. This applies to code coverage, lint scores, test pass rates, and any other measured quality gate.

- **Do not exclude files, suppress warnings, or weaken checks** just to hit a target. Only exclude things that genuinely shouldn't be measured (generated code, type declarations, etc.) — every exclusion needs a clear justification.
- **If a gate fails, fix the underlying issue** (add tests, fix the warning) or **lower the threshold honestly** to reflect the current state. A threshold that matches reality and ratchets upward over time is better than one that looks good on paper.
- **Prefer failing CI with accurate data** over green CI that hides gaps.
- Do not write tests just to increase coverage numbers. Every test must assert something meaningful about behaviour.
- Do not add test-only code paths to production code (`if TESTING:`, `# pragma: no cover` to hide real logic, etc.)
- If adding a test would require more complexity than the code it covers, skip it. Document why with a comment in the test file if non-obvious.

## Testing philosophy

- **Test-first for bug fixes** — write a failing test first that reproduces the issue, then fix the code. Never fix first and write a passing test after.
- **Test meaningful behaviour** — formatting correctness, path derivation, input validation, security boundaries.
- Use `tmp_path` fixture for file system tests
- Use `@pytest.mark.parametrize` for parameterised cases
- Use `pytest.raises()` for expected exceptions
- Plain functions + `assert`; no unittest classes

## Do not commit

- Files containing credentials or passwords
- `.env` files
- Database or index files
