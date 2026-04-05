---
name: pnpm-monorepo-trusted-publishing
description: Use this skill when setting up, validating, or debugging npm package release pipelines for pnpm workspace monorepos, especially with GitHub Actions, npm Trusted Publisher, provenance, scoped public packages, pack/publish tarball flows, and CI failures around workspace dependencies, Vue/TypeScript monorepo wiring, or npm release metadata.
---

# Pnpm Monorepo Trusted Publishing

## Overview

Use this skill for pnpm workspace monorepos that publish one or more packages to npm.
It is optimized for GitHub Actions + npm Trusted Publisher + provenance, with a strong focus on making release pipelines testable without burning version numbers.

## Quick Start

1. Inspect the monorepo shape first.
   Check root `package.json`, workspace config, package `package.json` files, release scripts, and workflow files.
2. Separate validation from publishing.
   Keep a normal CI workflow for install/lint/typecheck/test/build/dry-run, and reserve real publishing for tags or an explicit manual release path.
3. Treat workspace dependency rewriting as a packaging concern.
   For pnpm monorepos, prefer `pnpm pack` to produce the tarball that rewrites `workspace:^` / `workspace:*`.
4. Treat provenance as a publish concern.
   For GitHub Actions trusted publishing, publish the packed tarball with `npm publish <tgz> --provenance`.
5. Validate the tarball, not just the source tree.
   Unpack the generated `.tgz` and inspect the published `package.json`, especially `dependencies`, `repository`, `homepage`, `bugs`, and `exports`.

## Recommended Workflow Shape

For CI:

- `pnpm install --frozen-lockfile`
- `pnpm lint`
- `pnpm typecheck`
- `pnpm test`
- `pnpm build`
- optional docs build
- `DRY_RUN=1 pnpm release:ci`

For release:

- Trigger from a tag or explicit manual release workflow
- Use GitHub-hosted runners
- Grant `id-token: write`
- Keep package manager operations in pnpm
- Use npm only for the final provenance-aware publish step

## Release Script Pattern

Use this pattern when the repo is a pnpm workspace monorepo:

1. Read each publishable package version from its local `package.json`.
2. Skip versions that are already on npm.
3. In local/manual non-GitHub runs:
   Use `pnpm publish --access public --no-git-checks` when needed.
4. In GitHub Actions:
   Run `pnpm pack --pack-destination <temp-dir>`
   Publish the generated tarball with `npx --yes npm@11.5.1 publish <tarball> --access public --provenance`

This split avoids two common failures:

- `npm publish` from source can leak `workspace:*` dependencies into the published package
- upgrading global npm inside GitHub runners can be brittle

## Metadata Requirements

Before provenance publishing, ensure every published package has correct metadata in its own `package.json`:

- `repository`
- `homepage`
- `bugs`
- `publishConfig.access`

`repository.url` must match the GitHub repository used by provenance verification.

For monorepos, include `repository.directory` for each package when applicable.

## Trusted Publisher Checklist

Trusted Publisher is configured per package, not just per scope or repo.

For each published package:

- configure the exact GitHub owner or org
- configure the exact repository name
- configure the exact workflow filename
- only use `environment` if the workflow actually uses one

When debugging trusted publishing, verify the package-level configuration for every package being released.

## CI-Specific Monorepo Pitfalls

If the repo contains frontend or Vue packages, do not assume local success means CI will resolve workspace imports the same way.

Typical fixes:

- add explicit `resolve.alias` in `vitest.config.ts` for cross-workspace source imports used before build
- add explicit Vue parser configuration for ESLint if `.vue` parsing is unstable in CI
- add `@types/node` and `"types": ["node", ...]` when configs import `node:*` modules

## Troubleshooting Reference

Read [references/troubleshooting.md](references/troubleshooting.md) when:

- npm returns `E404`, `E422`, or provenance validation errors
- published tarballs differ from local expectations
- CI only fails for lint/typecheck/test in workspace packages
- tag-driven release behavior is surprising or too expensive to debug
