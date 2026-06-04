# Running isomorphic-git under Bare

isomorphic-git runs under Bare unmodified. One configuration requirement:
use the **ESM build**, not the CJS build.

## Why ESM

Bare resolves the `"node"` export condition by default. The `"node"` entry
in package.json points to the CJS build (`index.cjs`), which hard-requires
Node's `crypto` module. Bare doesn't have `crypto`.

The ESM build (`index.js`, the `"default"` export) falls back to `sha.js`
for hashing — pure JavaScript, no native crypto needed. All other
dependencies (pako, diff3, etc.) are also pure JS.

## Usage with bare-fs

```js
import git from './index.js'  // ESM entry — not the "node" condition
import fs from 'bare-fs'

await git.init({ fs, dir: '/path/to/repo' })
await git.commit({
  fs,
  dir: '/path/to/repo',
  message: 'initial',
  author: { name: 'test', email: 'test@test' }
})
const log = await git.log({ fs, dir: '/path/to/repo' })
```

## Usage with Hyperdrive

```js
import git from './index.js'
import { createFs } from './hyperdrive-fs.js'

const fs = createFs(drive)
await git.init({ fs, dir: '/' })
```

The Hyperdrive fs adapter maps isomorphic-git's fs interface to
Hyperdrive v11+ operations. See `hyperdrive-fs.js`.

## Pluggable fs

isomorphic-git takes an `fs` parameter on every call. Three
implementations:

- **bare-fs** — OS filesystem under Bare. Standard git, fully compatible.
- **Hyperdrive adapter** (`hyperdrive-fs.js`) — P2P storage. Git objects
  as Hyperdrive entries, individually addressable and replicable.
- **Node fs** — standard Node.js (for testing outside Bare).

Same git operations, same object model, different storage backends.

## Proven

Probe: `isomorphic-git-under-bare` (spl6 POC phase-5). init, add, commit,
log, working-tree reconstruction from `.git` — all on `bare-fs`, on
distroless Bare, 129MB image. All-pure-JS deps. `Buffer` is a Bare global.

Gotcha pinned: the ESM build is the only requirement.
