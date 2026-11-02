# yang

yang is a maintained fork of [YAPF](https://github.com/google/yapf), Google's
Python formatter, that formats every piece of Python 3.14 syntax.

YAPF 0.43.0 (November 2024) is the last release and the project is no longer
maintained. It crashes with `IndexError: tuple index out of range` on PEP 695
generics, PEP 701 f-strings and PEP 750 t-strings, rewrites `except*` as
`except *`, and has a set of older bugs that produce invalid or
non-idempotent output. yang fixes them and keeps everything else the same: of
the 101,308 formatting runs that YAPF 0.43.0 gets right across CPython's
`Lib/`, Home Assistant, mypy, pydantic and the black and ruff test suites,
yang's output is byte-identical in 101,146. The other 162 are deliberate:
`except *` becomes `except*`, and two comment placements are fixed.

The formatting engine, the style knobs, the command line and the library API
are YAPF's. Only the names changed (see [Migrating from YAPF](#migrating-from-yapf)).

## Install

```bash
uv tool install git+https://github.com/CarterPerez-dev/yang@v1.0.0
```

This installs the `yang` and `yang-diff` commands. yang runs on Python 3.7
through 3.14 and formats code that uses any syntax up to Python 3.14,
regardless of the Python version it runs on.

The distribution is named `yang-fmt` because `yang` is taken on PyPI. The
command and the import name are `yang`.

## Usage

```bash
yang -i -r src tests          # format in place, recursively
yang --diff file.py           # print a diff instead of changing the file
yang -i --style pep8 file.py  # pick a predefined style
git diff -U0 --no-color --relative HEAD^ | yang-diff -i   # format only changed lines
```

Every YAPF flag works unchanged; `yang --help` lists them.

As a library:

```python
from yang.yanglib.yang_api import FormatCode

formatted, changed = FormatCode("def f[T](x: T) -> T: return x\n",
                                style_config="pep8")
```

## pre-commit

```yaml
repos:
  - repo: https://github.com/CarterPerez-dev/yang
    rev: v1.0.0
    hooks:
      - id: yang
```

A `yang-diff` hook that formats only the lines changed in a commit is also
available.

## Configuration

yang reads its style from, in order:

1. `--style` on the command line
2. `.style.yang` in the file's directory or any parent directory
3. a `[yang]` section in `setup.cfg`
4. a `[tool.yang]` table in `pyproject.toml`
5. `~/.config/yang/style`
6. the `pep8` style

Predefined styles: `pep8`, `google`, `facebook`, `yang` (the style YAPF called
`yapf`; `based_on_style = yapf` is still accepted).

Files are excluded with `.yangignore` or a `[tool.yangignore]` table in
`pyproject.toml`, using the same glob syntax as YAPF.

Formatting can be switched off for a region with `# yang: disable` and
`# yang: enable`. The YAPF spellings `# yapf: disable` / `# yapf: enable` and
`# fmt: off` / `# fmt: on` are honored too, so existing code keeps its
disabled regions.

The full list of style knobs is in [docs/KNOBS.md](docs/KNOBS.md).

## Migrating from YAPF

| YAPF | yang |
| --- | --- |
| `yapf` / `yapf-diff` commands | `yang` / `yang-diff` |
| `.style.yapf` | `.style.yang` |
| `[yapf]` in `setup.cfg` | `[yang]` |
| `[tool.yapf]` in `pyproject.toml` | `[tool.yang]` |
| `.yapfignore` / `[tool.yapfignore]` | `.yangignore` / `[tool.yangignore]` |
| `~/.config/yapf/style` | `~/.config/yang/style` |
| `import yapf`, `yapf.yapflib.yapf_api` | `import yang`, `yang.yanglib.yang_api` |
| `YapfError` | `YangError` |
| pre-commit hook id `yapf` | `yang` |

Config file contents (the `[style]` knobs) do not change.

## What yang adds to YAPF 0.43.0

New syntax, formatted under every style:

- PEP 695 type parameters on functions and classes, and the `type` statement.
  Type parameter lists are split like parameter lists, including the magic
  trailing comma.
- PEP 696 type parameter defaults.
- PEP 701 f-strings: nested quotes of the same kind, replacement fields that
  span lines, comments and backslashes inside replacement fields. f-strings
  are kept byte for byte.
- PEP 750 template strings (t-strings).
- PEP 758 `except A, B:` and `except* A, B:` without parentheses.
- PEP 654 `except*` (no longer rewritten as `except *`).
- PEP 646 starred expressions in subscripts and annotations, including
  `x[*a or b]`, and starred expressions after augmented assignment.
- `await` and `async for` in generator expressions outside `async def`.
- Identifiers with combining marks and other characters PEP 3131 allows.

Older bugs fixed (each one produced invalid code, changed the program,
crashed, hung, or was not idempotent in YAPF 0.43.0):

- Parse errors are reported with their position instead of `IndexError`.
- `6 .real` no longer becomes `6.real`.
- `match`, `case` and `type` used as names next to soft-keyword statements.
- Statements split from a `;` are no longer glued together.
- Comments containing `\f`, `\v` or Unicode line separators no longer hang or
  break indentation.
- Form feeds and backslash-only lines in indentation.
- Several non-idempotent blank-line rules around comments.
- `RecursionError` on deeply nested expressions that CPython accepts
  (Python 3.12+).
- Grammar cache collisions between installed versions.
- A backslash right after a compound statement's colon (`else: \`) no longer
  grows whitespace on every run.
- Backslash-only lines inside indented blocks no longer crash or produce
  invalid code.
- `await []()`, `await (yield x)` and `await ...` outside `async def`.
- Long `X | Y` union annotations are formatted the same way on every run.
- Spacing and comments after strings continued with a backslash are kept on
  lines that are not reformatted.

## How it was verified

- Differential tokenizer test against CPython 3.14's own tokenizer: every
  string, f-string, t-string and identifier span matches on 27,485 real files
  and 50,000 generated f-strings and t-strings.
- AST equivalence and idempotency under pep8, google, facebook and a
  45-knob custom style on CPython 3.14's `Lib/`, Home Assistant, mypy,
  pydantic and the black and ruff formatter test suites (27,544 files,
  110,176 runs): zero crashes, zero AST changes, zero timeouts, and one
  non-idempotent result that YAPF 0.43.0 has too (see below). YAPF 0.43.0
  crashes on 7,194 of those runs, breaks the program on 141 and is
  non-idempotent on 53.
- Byte-for-byte comparison with YAPF 0.43.0 on every run that 0.43.0 formats
  correctly: 101,146 of 101,308 identical, the rest deliberate (see above).
- Negative controls: dropping a name, changing a number or changing comments
  on every pass is caught by the checks above on 100% of the files where the
  change applies.
- Golden tests for each new syntax feature under all four styles.
- The test suite passes on Python 3.7 through 3.14.

## Known limitations

- Under the google and facebook styles, a comment on its own line between a
  dictionary key and its value can move on the second run. YAPF 0.43.0 does
  the same.
- With `--lines`, a backslash-only line at column 0 before an indented line
  outside the requested range becomes a blank line. This shape only exists
  in formatter test suites.

## Credits and license

yang is a modified version of [YAPF](https://github.com/google/yapf),
Copyright Google Inc. and the YAPF authors, licensed under the Apache License,
Version 2.0. The vendored lib2to3 parser is Copyright the Python Software
Foundation. All credit for the formatter itself belongs to the YAPF authors.
See [LICENSE](LICENSE) and [NOTICE](NOTICE).
