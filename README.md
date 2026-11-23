# renovate-uv-bumpversions-repro

Minimal reproduction: `bumpVersions` for the `pep621` manager in a uv project bumps `project.version` in
`pyproject.toml`, but the root package entry in `uv.lock` keeps the old version.

Setup: `example` 0.1.0 with one outdated dependency (`six==1.16.0`), a committed `uv.lock`, and a
`bumpVersions` rule (patch) on `pyproject.toml` in `renovate.json`.

## Expected

The Renovate branch that updates `six` has `project.version = "0.1.1"` and a `uv.lock` whose `example`
package is also `0.1.1`, so `uv lock --check` passes.

## Actual

`pyproject.toml` says `0.1.1`, `uv.lock` still says `0.1.0`:

```text
$ uv lock --check
error: The lockfile at `uv.lock` needs to be updated, but `--check` was provided. ...
```

Cause: `processBranch` runs the artifact update (`uv lock --upgrade-package six`) before `bumpVersions`, so
`uv lock` never sees the bumped version.
