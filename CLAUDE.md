# Shared instructions

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
