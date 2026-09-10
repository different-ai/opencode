# OpenWork engine patches

Patches applied on top of the upstream `anomalyco/opencode` tag when
`engine-release.yml` builds a `v<upstream>-openwork.N` release.

## Branch layout

- `dev` — pristine mirror of `anomalyco/opencode#dev` (fast-forwarded daily by
  `upstream-sync.yml`; never commit here).
- `openwork-dev` — default branch: upstream `dev` + this directory + the
  OpenWork workflows. Patches and workflow changes land here via PRs.

## Adding a patch

1. Branch off the upstream tag you target: `git checkout -b fix v1.16.2`
2. Commit your fix with a clear message.
3. Export it: `git format-patch -1 --zero-commit -o openwork/patches/`
4. Commit the `.patch` file to `openwork-dev` and re-run `engine-release`
   (bump the `-openwork.N` suffix).

Patches are applied with `git am` in filename order. Keep them small,
upstream them when possible, and delete them once upstream ships the fix.

## Parent-first cancellation

`0001-fix-opencode-interrupt-parent-before-cancelling-task.patch` targets
upstream `v1.18.18`. It interrupts a parent runner before cancelling its task
jobs, preventing a waiting foreground task from waking an uncancelled parent
and issuing another provider request during Stop. The job sweep still runs
when no parent runner exists and still includes recursive descendants.

The patch includes a regression in `test/tool/task.test.ts` using the real
runner and background-job services, a readiness barrier, recursive children,
and an unrelated job. The test fails against unpatched `v1.18.18` and passes
with the patch. Existing no-runner cancellation tests remain enabled.

Local verification from `packages/opencode`, using Bun 1.3.14:

```sh
bun test test/tool/task.test.ts test/background/job.test.ts test/effect/runner.test.ts
bun typecheck
OPENCODE_VERSION=1.18.18-openwork.parent-first.84d1dc1 bun run script/build.ts --single --skip-install --skip-embed-web-ui
```

The local binary is a verification candidate, not a published release. Release
dispatch and the consuming OpenWork pin update must wait for PR approval and
the native Stop journey results. Keep `upstream_ref` at `v1.18.18` when building
this patch; a different upstream base requires revalidation.

## Cutting a release manually

Actions → engine-release → run with `upstream_ref` (e.g. `v1.16.2`) and
`version` (e.g. `1.16.2-openwork.1`).

## Consumption

The OpenWork repo pins `opencodeVersion` (and repo) in `constants.json`;
release assets keep upstream naming (`opencode-darwin-arm64.zip`, ...).
