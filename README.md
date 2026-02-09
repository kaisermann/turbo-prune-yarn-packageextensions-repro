# `turbo prune` orphaned lockfile entries repro

Minimal reproduction for a `turbo prune` bug where orphaned lockfile entries from Yarn's built-in `packageExtensions` (`plugin-compat` database) are left in the pruned `yarn.lock`, causing `yarn install --immutable` to fail with `YN0028`.

Created from `npx create-turbo@canary -e with-shell-commands`.

## Changes from the template

1. Upgraded from Yarn 1 to Yarn 4.5.0 (`packageManager` + `.yarnrc.yml` with `nodeLinker: node-modules`)
2. Updated workspace dependency specifiers to `workspace:*`
3. Added `got@^9.6.0` as a dependency of `app-b` (triggers Yarn's built-in `packageExtensions` to inject `@types/keyv` and `@types/responselike`)

## Reproduction steps

```bash
git clone https://github.com/kaisermann/turbo-prune-yarn-packageextensions-repro
cd turbo-prune-yarn-packageextensions-repro
yarn install
yarn turbo prune app-a --out-dir=out
cd out
yarn install --immutable   # FAILS with YN0028
```

### Expected

`yarn install --immutable` succeeds in the pruned output.

### Actual

```
➤ YN0085: │ - @types/keyv@npm:3.1.4, @types/responselike@npm:1.0.3
➤ YN0028: │ The lockfile would have been modified by this install, which is explicitly forbidden.
```

`@types/keyv` and `@types/responselike` are orphaned in the pruned lockfile — they were injected by Yarn's `plugin-compat` as dependencies of `got@9.6.0`, but `turbo prune` removed `got` (since `app-a` doesn't depend on it) without removing these extension-injected entries.
