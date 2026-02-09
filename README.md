# `turbo prune` leaves orphaned lockfile entries — minimal repro

Created from `npx create-turbo@canary -e with-shell-commands`, then minimized to two workspaces:

- **`app-a`**: empty workspace (prune target)
- **`app-b`**: depends on `got@^9.6.0`

`got@9.6.0` triggers Yarn's built-in `packageExtensions` ([plugin-compat](https://github.com/yarnpkg/berry/blob/master/packages/yarnpkg-extensions/sources/index.ts)), which injects `@types/keyv` and `@types/responselike` into the lockfile. When `turbo prune` removes `app-b` and its deps, it leaves these entries orphaned.

## Reproduce

```bash
yarn install
yarn turbo prune app-a --out-dir=out
cd out
yarn install --immutable   # FAILS with YN0028
```
