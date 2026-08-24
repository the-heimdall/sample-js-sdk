# @the-heimdall/sample-sdk

A throwaway package used to rehearse publishing to npm from GitHub Actions, before
applying the same pipeline to the real `@joyfill/components` SDK.

## How releasing works

Nothing publishes from a laptop. A release happens when you push a version tag:

```bash
npm version 0.0.2 -m "Release v%s"
git push origin main --follow-tags
```

The tag push triggers `.github/workflows/release.yml`, which verifies the tag matches
`package.json`, installs, lints, and runs `npm publish`.

Versions containing a `-` (e.g. `0.0.2-rc1`) publish under the `beta` dist-tag instead
of `latest`, so a release candidate never becomes the default install.

## The two workflows

| File | Fires when | What it does | Publishes? |
|---|---|---|---|
| `.github/workflows/ci.yml` | Every PR, every push to `main` | install → lint | No |
| `.github/workflows/release.yml` | Push of a `v*` tag | verify tag → install → lint → publish | Yes |

## How authentication works

`.npmrc` is committed but contains **no secret** — only a reference to an environment
variable:

```
//registry.npmjs.org/:_authToken=${NPM_TOKEN}
```

That same file works in two places because each supplies `NPM_TOKEN` differently:

- **CI** — GitHub injects it from the `NPM_TOKEN` repository secret
- **Your laptop** — 1Password injects it for a single command

## Working locally

Secrets are never written to disk. `.env.op` holds 1Password *pointers*, not values:

```bash
# check the token resolves
op run --env-file=.env.op -- node -e "console.log(process.env.NPM_TOKEN ? 'token loaded' : 'NOT loaded')"

# rehearse a publish without touching the registry
op run --env-file=.env.op -- npm publish --dry-run
```

## Setup required once

- A `NPM_TOKEN` repository secret (npm Granular Access Token, read+write, scoped to `@the-heimdall`)
- A 1Password item `npm-publish-token` in the `Joyfill` vault, holding the same token
