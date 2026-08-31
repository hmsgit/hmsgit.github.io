# hmsgit.github.io

This repository publishes [hmsgit.github.io](https://hmsgit.github.io). It holds no
content of its own — the site lives in
[hmsgit/singularity](https://github.com/hmsgit/singularity) (Astro), and the
workflow here checks that repo out, builds it, and deploys the result to GitHub
Pages.

## One-time setup

**Settings → Pages → Source: GitHub Actions.** Without it the build succeeds but
nothing gets published.

## What triggers a deploy

| Trigger | When |
| --- | --- |
| `workflow_dispatch` | manually, from the Actions tab |
| `push` | a commit lands on `master` here |
| `repository_dispatch` | immediately, if singularity signals (below) |

## Deploying as soon as singularity changes

Deploys are manual by default — no scheduled runs, no wasted Actions minutes. For
push-to-live instead, give singularity a way to poke this repo:

1. Create a fine-grained personal access token with **Contents: read & write** on
   `hmsgit/hmsgit.github.io`.
2. Save it in the singularity repo as the secret `PAGES_DISPATCH_TOKEN`.
3. Add this workflow to singularity as `.github/workflows/notify-pages.yml`:

```yaml
name: Notify pages
on:
  push:
    branches: [master]
jobs:
  notify:
    runs-on: ubuntu-latest
    steps:
      - run: |
          curl -sS -X POST \
            -H "Authorization: Bearer ${{ secrets.PAGES_DISPATCH_TOKEN }}" \
            -H "Accept: application/vnd.github+json" \
            https://api.github.com/repos/hmsgit/hmsgit.github.io/dispatches \
            -d '{"event_type":"source-updated"}'
```

## How it reads the private source

singularity is private, so the checkout uses a **read-only deploy key**: the
public half is registered on singularity, the private half lives here as the
`SOURCE_SSH_KEY` secret. That key can read that one repository and do nothing
else. To rotate it, generate a new ed25519 pair, replace the deploy key on
singularity, and update the secret here.
