# Code style

Write code that a reader understands at the first read. This page applies to every Dende repository.
The examples are Python and SQL. The web app follows the same rules; its `AGENTS.md` gives TypeScript examples.
Each repository's `AGENTS.md` names its reference code. Copy that structure and follow this page for the
details. Older code can break these rules. Do not copy it.

## Follow the known standards

Python code follows these standards. The sections below add the Dende rules. Where a standard and this page
differ, this page decides.

- [PEP 8](https://peps.python.org/pep-0008/), [PEP 257](https://peps.python.org/pep-0257/) and
  [PEP 20](https://peps.python.org/pep-0020/).
- The [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html), with two differences: a line
  has at most 120 characters, and a module can import names, not only modules.
- YAGNI: build nothing that no acceptance example needs.
- The rule of three: extract shared code when a third place needs it.
- "Parse, don't validate": parse an untyped value once, where it enters the code, into a typed value.
- Tests: Arrange-Act-Assert. Test the behavior, not the implementation.
- In a review, name over-engineering with the code smells of Fowler's "Refactoring": Speculative Generality,
  Middle Man, Lazy Element.

Do not apply the tiny functions of "Clean Code", or the SOLID principles and the Gang of Four patterns,
by default. They add wrappers, interfaces and factories that no acceptance example needs.

## Put code in its layer

- `domain/`: values and rules. No I/O, no framework, no clock.
- `application/`: use cases. A use case defines the ports that it needs, as `Protocol` classes.
- `adapters/`: HTTP, SQL, files and queues. An adapter implements a port and translates values.
  It holds no business rule.
- Dependencies point inward: adapters, then application, then domain. Add a port only for a real external boundary.
- A layer folder or file exists only when it holds code that its layer owns, such as a value, a rule, a port,
  an order of steps, a translation or the wiring. Do not add one that only passes a call on.

```python
# Bad: business code depends on the web framework
from fastapi import Request

# Good: the use case defines the port that it needs ...
class ChapterReader(Protocol):
    async def chapter(self, chapter_id: UUID) -> Chapter | None: ...

# ... and an adapter implements it
class PostgresChapterReader:
    async def chapter(self, chapter_id: UUID) -> Chapter | None: ...
```

## Put each Python import at the top of the module

- Write every import at the top of the module, after the module docstring. Never write one in a function,
  a method, a class or a test. ruff rule `PLC0415` checks this.
- When two modules need each other at run time, move the code that both need into a third module.
  Do not break the cycle with an import in a function.
- An `if TYPE_CHECKING:` block at the top can hold the imports that only type annotations use.

```python
# Bad: the import hides a dependency inside the function
def parse_cursor(cursor: str) -> Cursor:
    import base64

# Good: the first lines of the module show each dependency
import base64
```

## Write the obvious version

- One idea per line. No tuple assignment of unrelated values and no unpacking tricks.
- Choose the version that a reader understands without stopping, even when it is longer.

```python
# Bad
payload, signature = (decode(part) for part in cursor.split("."))

# Good
parts = cursor.split(".")
if len(parts) != 2:
    raise InvalidCursorError
payload, signature = decode(parts[0]), decode(parts[1])
```

## Use the words of the domain

Name each thing with the words of the issue and of the module docstrings. No `data`, `info`, `helper`,
`manager`, `util` or one-letter names, except a type parameter such as `T`. Return a typed value, not a bare `dict`.

```python
# Bad
async def get_leaderboard_data(r: Redis, challenge_id: str, user_id: str) -> dict: ...

# Good
async def leaderboard_window(cache: Redis, challenge_id: str, user_id: str) -> LeaderboardWindow: ...
```

## Name each setting

Pass settings, numbers and flags as keyword arguments. Shared settings classes use `kw_only=True`,
so a positional call fails.

```python
# Bad
limit = RateLimit("chapter-read", 10, 20, 7, True)

# Good
limit = RateLimit(name="chapter-read", rate=10, burst=20, concurrency=7, fail_closed=True)
```

## Handle only the errors that you can act on

- No broad `except`. No silent fallback value that hides an error.
- No check for a case that the types or the code already exclude.

```python
# Bad
try:
    return int(match.group(1))
except Exception:
    return None

# Good: the pattern matched only digits
return int(match.group(1))
```

## Document in the code

- The only documentation files of a repository are one short `README.md` at its root, the instruction files,
  the PR template and `runbooks/`, with one file for each production procedure.
- The acceptance tests state the behavior, the docstrings state the contract and the comments state why.
  The issue holds the decisions and the history.
- Each Python module and each `__init__.py` starts with a docstring that says what it owns and its rules.
- A public function or class has a docstring when, and only when, its name and types do not say everything:
  a rule, a unit or a side effect. Write `Raises:` only for an exception that the caller must handle.
  Write no `Args:` and no `Returns:`.
- A comment says why the code is so, never what it does. For a "what", rename or simplify the code.
  A comment has at most two lines. A longer reason goes in the docstring.
- A comment or a docstring makes sense to a reader who has only the file. It never names the task, the issue,
  the PR, the review, the plan or the conversation, and it never tells how the code was before or what changed.
  It describes this code, not its callers.
- Write full sentences in ASD-STE100. No "I", no "we" and no TODO: open work is a Linear issue.

```python
# Bad: the comment needs the task to make sense, and it tells the history
# TECH-5088: now we read each page, as the review asked.

# Good: the comment says why, to any reader of the file
# The head filter of GitHub finds no PR in a fork, so the function reads each page.
```

## Lay out SQL by clause

One clause per line. A space after each comma and around each operator. `AS` for each alias.
Each extra condition on its own line. The `dende-sql` skill holds the example and the query rules.

## Test behavior, once

- The acceptance examples are the main tests: one test each, through the real interface (HTTP for a route).
  Name each test after the rule that it proves.
- Write a unit test only for pure logic with many cases. Put the cases in a table (`pytest.mark.parametrize`).
- Assert results, not call sequences. Exception: for a security order rule, such as "no read before
  the cursor check", a test may assert that a port was not called.
- A test of a race or of the wiring asserts the stored result. Run the real composition, put the competing write
  between the steps, and assert the row. Do not assert SQL text or a constructor argument.
- Do not repeat a behavior that a higher-level test already proves.

```python
# Bad
assert calls == [("access", query), ("items", access, None, 3), ("encode", scope, last_id)]

# Good
assert page.items == (first, second)
assert page.has_next_page is True
```

## Tools

- ruff, `mypy --strict` and import-linter check the rules that a tool can check. dende-cicd owns their central
  configuration, and its QA runs it on each repository. A repository can add rules, never remove them.
- A rule of this page holds also where no tool reports it. The formatter decides the layout.
- Do not add `noqa`, `type: ignore` or `pragma: no cover`.
