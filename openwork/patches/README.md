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

## Cutting a release manually

Actions → engine-release → run with `upstream_ref` (e.g. `v1.16.2`) and
`version` (e.g. `1.16.2-openwork.1`).

## Consumption

The OpenWork repo pins `opencodeVersion` (and repo) in `constants.json`;
release assets keep upstream naming (`opencode-darwin-arm64.zip`, ...).
