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
  `TAGS` file with at least one valid tag; anything else is ignored, and the two
  non-demo repos that do carry tags (`dungeon-master`, `template`) are skipped
  by name.
- **Test** the demos against the active morloc version. This needs no `.git`,
  so `--repos-dir` can point at a local checkout to test unpushed demos.
- **Release**: test the whole corpus (honest builds, compiler gate in force) and
  bundle the passing demo sources into `demos.tar.gz` + `manifest.tsv`. A demo
  that fails and is not on `allow-fail` halts the release; a demo on `allow-fail`
  is excluded rather than blocking.

The release tarball is the source of truth for "which demos work on which morloc
version": because it is built with the gate in force, a demo can never be
bundled for a version it does not actually build and pass on.

## Commands

```
dungeon-master list    [--repos-dir DIR] [--tag T]
dungeon-master clone   [--repos-dir DIR] [--tag T]
dungeon-master update  [--repos-dir DIR] [--tag T]
dungeon-master test    [--repos-dir DIR] [--tag T]
dungeon-master release [--repos-dir DIR] [--bundle] [--out DIR] [--morloc-version V]
```

`--repos-dir` is the directory holding one folder per demo, relative to the cwd;
it defaults to `repos/` beside the script, which is where `clone` puts the org's
repos. From a workspace that holds the demos as siblings:

```
python3 dungeon-master/dungeon-master test --repos-dir .
```

`release` gates the corpus (nonzero exit on any non-`allow-fail` failure); with
`--bundle` it also writes the tarball + manifest. `--morloc-version V` asserts
the active compiler is V. `release` requires every demo to be a git repository,
since the manifest cites and the bundle archives the tested commit.

## Release CI

`.github/workflows/release.yml` runs the demos in a native `mim` environment
(`mim new --engine none`) on both Linux and macOS. Each runner gates; a red leg
on either OS cancels the release. Only the Linux leg bundles (the bundle is
OS-agnostic source), and a publish job attaches the tarball to a
`demos-<version>` GitHub release.

Only a `v*` tag publishes: it gates against the latest compiler release and
attaches the bundle. A push to `main` only gates, as does a manual dispatch
(which can name a compiler version), so the corpus is shown green before a tag
changes what `mim demos` pulls. To publish, after every demo is pushed and the
corpus passes locally:

```
git tag v$(cat VERSION) && git push origin v$(cat VERSION)
```

`VERSION` is this tool's version; the published release is named after the
compiler version the demos were tested against, so the two numbers are
unrelated.

## Demo repository contract

Every demo repo has a fixed layout so `dungeon-master` can drive them
identically: a `Makefile` exposing `build` / `test` / `clean`, a `TAGS` file
(one tag per line, `[A-Za-z0-9_-]+`), a `package.yaml`, and the morloc program
plus its expected test output. See the [`template`](../template) repo for the
canonical layout.
