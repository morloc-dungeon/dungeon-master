# dungeon-master

Admin tooling for the [`morloc-dungeon`](https://github.com/morloc-dungeon)
organization -- a curated collection of small morloc example programs used for
teaching and as an external compiler regression corpus.

**This is an admin/curation tool, not the tool a learner runs.** It clones the
demo repos, tests them against a morloc version, and bundles the passing ones
into a single release tarball. The user-facing path for pulling demos is a
separate concern and will be folded into `mim`, consuming the release artifacts
produced here.

The tool is written in Python (stdlib only).

## What it does

- **Clone/update** every demo repo in the org. A repo is a demo iff it has a
  `TAGS` file with at least one valid tag; anything else (including this repo) is
  ignored.
- **Test** the demos against the active morloc version.
- **Release**: test the whole corpus (honest builds, compiler gate in force) and
  bundle the passing demo sources into `demos.tar.gz` + `manifest.tsv`. A demo
  that fails and is not on `allow-fail` halts the release; a demo on `allow-fail`
  is excluded rather than blocking.

The release tarball is the source of truth for "which demos work on which morloc
version": because it is built with the gate in force, a demo can never be
bundled for a version it does not actually build and pass on.

## Commands

```
dungeon-master list    [--tag T]
dungeon-master clone   [--tag T]
dungeon-master update  [--tag T]
dungeon-master test    [--tag T]
dungeon-master release [--bundle] [--out DIR] [--morloc-version V]
```

`release` gates the corpus (nonzero exit on any non-`allow-fail` failure); with
`--bundle` it also writes the tarball + manifest. `--morloc-version V` asserts
the active compiler is V.

## Release CI

`.github/workflows/release.yml` runs the demos in a native `mim` environment
(`mim new --engine none`) on both Linux and macOS. Each runner gates; a red leg
on either OS cancels the release. Only the Linux leg bundles (the bundle is
OS-agnostic source), and a publish job attaches the tarball to a
`demos-<version>` GitHub release. See [DESIGN.md](DESIGN.md) for the rationale.

## Demo repository contract

Every demo repo has a fixed layout so `dungeon-master` can drive them
identically: a `Makefile` exposing `build` / `test` / `clean`, a `TAGS` file
(one tag per line, `[A-Za-z0-9_-]+`), a `package.yaml`, and the morloc program
plus its expected test output. See the [`template`](../template) repo for the
canonical layout.
