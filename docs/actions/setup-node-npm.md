# `setup-node-npm`

Checkout + `setup-node` (from `.nvmrc` by default) + `npm ci` with caching, in one step. Use it inside your own jobs when [`node-ci.yml`](../workflows/node-ci.md) is not a fit.

```yaml
steps:
  - uses: domengabrovsek/github-actions/.github/actions/setup-node-npm@main
  - run: npm run build
```

## Inputs

| Input | Default | Description |
|-------|---------|-------------|
| `node-version-file` | `.nvmrc` | File that pins the Node version. Ignored when `node-version` is set. |
| `node-version` | `''` | Explicit Node version. Overrides `node-version-file`. |
| `install` | `true` | Run `npm ci` after setup. Set `false` to skip installing. |
| `install-args` | `''` | Extra args appended to the baseline `npm ci`. |
| `fetch-depth` | `1` | Commits to fetch. `0` fetches full history. |

## Notes

- The baseline install is `npm ci --ignore-scripts --no-audit --no-fund` (supply-chain safety + less CI noise). `install-args` is appended for repo-specific extras.
- Checkout runs through the [`checkout`](checkout.md) wrapper, so `actions/checkout` is pinned in a single place.
