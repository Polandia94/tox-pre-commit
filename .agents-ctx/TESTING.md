# Testing

Grounded in the maintainer's review history in other projects
(PR numbers below are from `ansible/pylibssh`), plus explicit
maintainer conventions carried over from other projects — not
guesses.

## Tests are proof, not decoration

New or changed behavior needs a test demonstrating it, covering the
cases actually touched (including edge cases like `None`). "I don't
think there's a proof that this works as intended without tests" is
a near-verbatim recurring review comment (e.g. PR #756, #786, #790,
#479, #669, #809). Missing test coverage is the single most common
reason PRs stall or get closed unmerged.

Justify new tests by what they actually verify, not by the
coverage number alone. Coverage still matters a lot here --
`.codecov.yml` targets 100%, and 100% coverage is a genuinely
useful self-check (e.g. it can catch tests that silently start
getting skipped in CI). The point is: don't game the number -- a
test that exercises a line without meaningfully checking behavior
doesn't count, passing or not. The same reasoning-over-metrics
standard applies to every change, not just tests: infra,
packaging, and runtime code all need well-reasoned,
architecturally sound justification, not just a passing check.

Deleting, commenting out, skipping, xfailing, or weakening a test
is never the agent's call -- see
[DONT_SHOOT_THE_MESSENGER.md](DONT_SHOOT_THE_MESSENGER.md).

## No test classes

Every test in `tests/` is a plain `def test_*` function. Don't
introduce `unittest.TestCase` subclasses or bare grouping classes.

Here: the only class in `tests/` is `EntryPointParam` in
`tests/importable_test.py`, a frozen dataclass holding
`parametrize` values -- data, not a test grouping.

## Prefer parametrize/fixtures over duplicated test logic

When two tests differ only in their input data, that's a
parametrized fixture or test, not two near-identical functions.
Precedent: PR #638 collapses two payload fixtures into one
parametrized fixture.

Loop-based checks inside a single test are generally a sign the
test should be parametrized instead. The one accepted exception is
`pytest-subtests` (the `subtests` fixture), for structured,
individually reported sub-assertions within one test.

## Real e2e over mocking

Don't introduce mocking as a substitute for exercising the real
behavior of the components involved.

Here: the integration tests use the `tox_project` fixture from
`tox.pytest` to create real tox projects in `tmp_path` and run
tox against them, with the plugin loaded. The one `monkeypatch`
use, in `tests/conftest.py`, sets `TOX_USER_CONFIG_FILE` to keep
the operator's own tox config out of the tests -- an environment
variable, i.e. a system boundary (see below).

## Design for testability

Prefer decoupling techniques -- dependency injection in particular
-- that let code be tested in isolation, over reaching for
monkeypatching. Monkeypatching (via `unittest.mock` or the
`monkeypatch` fixture) is only acceptable at real system
boundaries: the system clock/timezone, environment variables, or
swapping in a stub server. It's not a substitute for designing
components to be testable on their own.

## Test doubles for intercepted components

When end-to-end tests need to patch or intercept a real component
-- say, to withhold a server's replies -- the test double gets its
own directory in the tests' stub area, named after its purpose
rather than the library it's built on. Its `fixtures.py` module is
the entry point, loaded as a local pytest plugin.

- Entry points are imported eagerly at startup, so they must not
  import their component's optional dependencies at the module
  level. Import those inside a fixture through
  `pytest.importorskip()`, so that a missing one skips only the
  tests that need it. That's a declared optional dependency, not
  a way to dodge a failure.
- The stub area is excluded from the doctest collection, which
  would otherwise import those dependencies.
- Code running in child processes only gets its coverage recorded
  when those processes exit cleanly instead of being killed.

## Test layout

Test files are named `<topic>_test.py`, not `test_<topic>.py`.
Match the existing file when extending it.

Here: `tests/` is flat -- `smoke_test.py` (importability),
`importable_test.py` (entry-point exposure), and
`integration_test.py` (behavior through real tox runs).

## Tests usually land in the same commit as the behavior they cover

When a change adds or alters behavior, the test change normally
lands in the *same* atomic commit, not a follow-up -- there's no
evidence of strict test-first/TDD commit ordering as a project
ritual. That style of TDD is mainly useful for interactive
development and exploration, used in tandem with continuous
adjustment, not as a blanket rule here.

The one accepted exception: a PR that adds only a failing test,
marked `@pytest.mark.xfail`, can be merged on its own ahead of the
fix, to be resolved in a follow-up. Don't treat a merged `xfail` as
done, though -- it's a tracked gap, not a resolution. This
exception covers adding a *new* failing test; marking an existing
test `xfail` because it broke is silencing it.

## Assertions

Use `pytest.raises(..., match=...)` -- never a bare
`pytest.raises(SomeError)` -- so the assertion pins down which
failure is expected, not just its type.

## Filesystem fixtures

Use pytest's built-in `tmp_path` fixture; don't hand-roll
temp-directory setup/teardown.

## Type annotations on new tests

New test functions and fixtures need type annotations too --
parameters and return types -- same as any other new code. See
[PR_HYGIENE.md](PR_HYGIENE.md) for the full rule.

## Every test and fixture has a one-line docstring

Describing what it checks or provides, not restating the code,
e.g. `"""Isolate the tests from the user's tox configuration."""`.
Enforced by `darglint` in CI.
