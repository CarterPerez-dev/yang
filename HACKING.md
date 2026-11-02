# Hacking on yang

## Run the tests

```bash
uv run --python 3.14 --with . --with pytest python -m pytest -q yangtests
```

The suite is expected to pass on every Python from 3.7 to 3.14.

## Format yang with itself

```bash
uv run --with . yang -i -r yang yangtests third_party
```

## Golden tests for new syntax

`yangtests/testdata/modern_syntax/*.in` are inputs and
`*.{pep8,google,facebook,custom}.out` the expected output under each style.
After an intended formatting change, regenerate the `.out` files with
`yang` and review every diff before committing it.

## Release

1. Bump `yang/_version.py`.
2. Add the release to `CHANGELOG.md`.
3. Tag `vX.Y.Z` and push the tag.
