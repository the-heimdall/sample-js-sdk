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

There is **no npm token anywhere** — not in this repo, not in GitHub secrets, not on
your laptop beyond your own login.

CI publishes using **npm Trusted Publishing (OIDC)**. GitHub mints a short-lived
identity token scoped to this exact repository and workflow, and npm verifies it
against the trusted publisher configured on the package. Nothing long-lived exists
to leak.

This replaced token auth because npm is
[retiring 2FA-bypass tokens](https://github.blog/changelog/2026-07-31-restricting-npm-bypass-2fa-granular-access-tokens/):
such tokens lose direct publish capability entirely in January 2027.

> **Do not commit an `.npmrc` with an `_authToken` line for registry.npmjs.org.**
> A project-level `.npmrc` overrides your user-level `~/.npmrc`, so it silently
> breaks `npm login` for everyone working in the repo — and the resulting failure
> reports as `404 Not Found`, not `401`, because npm hides the existence of scoped
> packages you can't prove access to. This repo learned that the hard way.
>
> Committing scope lines for a *private* registry (e.g. `@fortawesome:registry=...`)
> is fine and normal — it's specifically an auth token for your publish target that
> causes the conflict.

## Working locally

```bash
npm install
npm run lint
npm publish --dry-run   # packages everything, touches no registry
```

If you ever need to publish by hand, `npm login` is all that's required.

## Setup required once

- A **trusted publisher** on the package at npmjs.com → Settings → Trusted Publisher:
  organization/user `the-heimdall`, repository `sample-js-sdk`, workflow `release.yml`
