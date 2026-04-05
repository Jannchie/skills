# Troubleshooting

## Package Publishes With `workspace:*`

Symptom:

- external install fails with workspace dependency errors

Likely cause:

- `npm publish` was run directly from source in a pnpm workspace package

Fix:

1. Run `pnpm pack`
2. Inspect the generated tarball `package.json`
3. Publish the tarball, not the raw workspace package source

## GitHub Action Uses the Wrong pnpm Version

Symptom:

- `pnpm/action-setup` complains about multiple versions of pnpm

Likely cause:

- workflow sets a `version:` value while root `package.json` also sets `packageManager`

Fix:

- keep one source of truth, usually root `packageManager`

## Vue SFC Lint Fails Only in CI

Symptom:

- `.vue` files report parse errors for TypeScript syntax

Likely cause:

- CI does not reliably resolve Vue ESLint parser chain from transitive dependencies

Fix:

- add direct dev dependencies for `vue-eslint-parser`, `eslint-plugin-vue`, and `@typescript-eslint/parser`
- add an explicit `**/*.vue` parser override in flat ESLint config

## `vue-tsc` Cannot Find `node:*`

Symptom:

- errors around `node:url`, `node:path`, or missing `node` types

Likely cause:

- the package tsconfig includes Vite config files but does not include Node types

Fix:

- add `@types/node`
- add `"node"` to `compilerOptions.types`

## Vitest Cannot Resolve Another Workspace Package

Symptom:

- tests fail before build with package export or entry resolution errors

Likely cause:

- test runner resolves package imports through published-style package metadata instead of workspace source

Fix:

- add explicit `resolve.alias` entries in `vitest.config.ts`
- point aliases to source entry files in sibling packages

## Trusted Publisher Returns `E404`

Symptom:

- npm says the package could not be found or you do not have permission

Likely cause:

- trusted publisher configuration does not match the actual GitHub workflow identity
- the package-level trusted publisher entry is missing

Check:

- package exists on npm
- trusted publisher is configured for each package
- GitHub owner or org matches exactly
- repository matches exactly
- workflow filename matches exactly

## Provenance Validation Returns `E422`

Symptom:

- npm rejects the sigstore provenance bundle

Common cause:

- `package.json.repository.url` is missing or does not match the GitHub repository in provenance

Fix:

- set `repository.url` to the canonical GitHub repository URL
- for monorepo packages, also set `repository.directory`
- repack and recheck the tarball before retrying

## Tag Releases Waste Versions During Debugging

Symptom:

- every workflow test burns another npm version

Fix:

- add a normal CI workflow that runs release dry-run on pushes and PRs
- make manual release default to dry-run
- reserve real publishing for tags or explicit dry-run=false manual execution
