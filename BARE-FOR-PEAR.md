# bare-for-pear/isomorphic-git

Fork of [isomorphic-git](https://github.com/isomorphic-git/isomorphic-git) with
a Hyperdrive fs adapter and Bare runtime documentation. Full isomorphic-git — all
porcelain and plumbing operations available.

**Upstream:** [github.com/isomorphic-git/isomorphic-git](https://github.com/isomorphic-git/isomorphic-git) (MIT)

## What This Fork Adds

1. **Hyperdrive fs adapter** (`hyperdrive-fs.js`) — maps isomorphic-git's 10-method
   fs interface to Hyperdrive v11+ operations. Plugs into the `fs` parameter.
2. **Bare runtime documentation** (`BARE.md`) — how to run isomorphic-git under
   Bare (use the ESM build, not CJS).

The upstream code is unmodified. This is not a barification — isomorphic-git is
pure JS and runs under Bare as-is once you import the ESM entry point.

## Dual Backend

Same git operations, two storage backends:

- **bare-fs** — standard OS filesystem. The correctness oracle.
- **Hyperdrive** — P2P storage. Git objects as drive entries, individually
  addressable and replicable across the swarm.

Selected by passing the appropriate fs implementation to isomorphic-git.

## Hyperdrive fs Adapter

`hyperdrive-fs.js` maps 10 fs methods to Hyperdrive v11:

| fs method | Hyperdrive operation | Notes |
|---|---|---|
| readFile | drive.get() | Returns Buffer; throws ENOENT if null |
| writeFile | drive.put() | |
| mkdir | no-op | Implicit directories |
| rmdir | no-op | Implicit directories |
| unlink | drive.del() | |
| stat | drive.entry({follow: true}) | Synthesised Stats with implicitDir flag |
| lstat | drive.entry({follow: false}) | |
| readdir | drive.readdir() | Stream collected to array |
| readlink | entry.value.linkname | |
| symlink | drive.symlink() | |

## Current State

**Proven.** Tested via spl6 POC probe (`isomorphic-git-on-hyperdrive`):

- Porcelain (init/add/commit/log): identical hashes on both backends
- Plumbing (writeBlob/writeTree/writeCommit): identical OIDs on both backends
- Full round-trip (bare-fs → Hyperdrive → bare-fs): 14 git objects, all identical
- Implicit directory handling confirmed (the implicitDir Stats fix)

**Not yet exercised:** merge, branch switching, tag operations, large repos,
concurrent access, symlink round-trip, error edge cases.

