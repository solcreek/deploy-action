# Creek Deploy Action

Deploy to the edge in seconds. No account required for sandbox previews.

[![Creek](https://img.shields.io/badge/creek-deploy-blue)](https://creek.dev)
[![License](https://img.shields.io/badge/license-Apache%202.0-green)](LICENSE)

## Quick Start

### Sandbox preview (no account needed)

```yaml
- uses: solcreek/deploy-action@v1
  with:
    dir: ./dist
```

Every PR gets a live preview URL. No signup, no API key.

### Production deploy

```yaml
- uses: solcreek/deploy-action@v1
  with:
    dir: ./dist
    token: ${{ secrets.CREEK_TOKEN }}
```

## Examples

### PR Preview with Comment

```yaml
name: Preview
on: pull_request

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci && npm run build

      - uses: solcreek/deploy-action@v1
        id: creek
        with:
          dir: ./dist

      - uses: actions/github-script@v7
        with:
          script: |
            github.rest.issues.createComment({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: context.issue.number,
              body: `**Creek Preview** — deployed in ${${{ steps.creek.outputs.duration }}}ms\n\n${{ steps.creek.outputs.url }}`
            })
```

### Production on Push to Main

```yaml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci && npm run build

      - uses: solcreek/deploy-action@v1
        id: creek
        with:
          dir: ./dist
          token: ${{ secrets.CREEK_TOKEN }}

      - run: echo "Deployed to ${{ steps.creek.outputs.url }}"
```

### Auto-Detect Framework

```yaml
# No dir needed — Creek detects your framework and build output
- uses: actions/checkout@v4
- run: npm ci && npm run build
- uses: solcreek/deploy-action@v1
```

Creek auto-detects `dist/`, `build/`, `out/`, or reads `creek.toml`.

### Demo Deploy (Connectivity Test)

```yaml
- uses: solcreek/deploy-action@v1
  with:
    demo: true
```

Deploys a sample page to verify Creek is reachable. Useful for CI smoke tests.

## Inputs

| Input | Required | Default | Description |
|-------|:--------:|---------|-------------|
| `dir` | No | auto-detect | Directory to deploy |
| `token` | No | — | Creek API key ([get one](https://app.creek.dev)). Omit for sandbox mode. |
| `demo` | No | `false` | Deploy a sample site |
| `protect` | No | — | Password-protect this deployment |

## Outputs

| Output | Description |
|--------|-------------|
| `url` | Deployed URL |
| `preview-url` | Preview URL (authenticated deploys only) |
| `sandbox-id` | Sandbox ID (sandbox mode only) |
| `deployment-id` | Deployment ID (authenticated deploys only) |
| `duration` | Deploy duration in milliseconds |
| `mode` | `sandbox`, `production`, or `demo` |

## Sandbox vs Production

| | Sandbox (no token) | Production (with token) |
|--|:------------------:|:----------------------:|
| Account required | No | Yes |
| TTL | 60 minutes | Permanent |
| Custom domain | No | Yes |
| Rate limit | 10/hr | Unlimited |
| Use case | PR previews, demos | Shipping to production |

## How It Works

This action runs `npx creek deploy --json` under the hood. The Creek CLI auto-detects your framework, collects build output, and deploys to [Cloudflare's edge network](https://creek.dev).

- **Sandbox mode**: Deploys to `*.creeksandbox.com` (no auth, 60 min TTL)
- **Production mode**: Deploys to `*.bycreek.com` or your custom domain

## Links

- [Creek](https://creek.dev) — Platform homepage
- [CLI Documentation](https://www.npmjs.com/package/creek) — Full CLI reference
- [GitHub](https://github.com/solcreek/creek) — Source code

## License

Apache 2.0
