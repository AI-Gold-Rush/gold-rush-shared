# gold-rush-shared

Canonical CI / AppSec configuration for the Gold Rush SaaS factory.

Consumed by:

- [`pool-pulse`](https://github.com/rich1edwards/pool-pulse) — Next.js web
- [`pool-pulse-mobile`](https://github.com/rich1edwards/pool-pulse-mobile) — React Native / Expo
- [`prospector`](https://github.com/10k-challenge/prospector) — Node + Python
- [`product-catalog`](https://github.com/10k-challenge/product-catalog) — docs/data
- [`gold-rush-ops`](https://github.com/rich1edwards/gold-rush-ops) — Python

## What's here

```
.github/workflows/
  security.yml          # reusable AppSec gate (workflow_call)
  dast-preview.yml      # reusable DAST baseline against Vercel preview URLs

security/
  gitleaks.toml         # 15+ Gold Rush-specific secret detectors
  osv-scanner.toml      # vulnerability allowlist policy
  semgrep/
    gold-rush-rules.yml # custom rules (Clerk, Stripe, Supabase, LLM)

actions/
  (placeholder for composite actions)
```

## How to consume

In each sub-repo, add a thin caller workflow at `.github/workflows/security.yml`:

```yaml
name: Security
on:
  pull_request:
  push:
    branches: [main]
  schedule:
    - cron: '0 4 * * *'

permissions:
  contents: read
  security-events: write
  pull-requests: write
  actions: read

jobs:
  security:
    uses: rich1edwards/gold-rush-shared/.github/workflows/security.yml@main
    with:
      stack: nextjs        # nextjs | react-native | python | mixed | docs
      enable_bandit: false
      enable_trivy: true
      enable_zap: false
    secrets: inherit
```

## Versioning

- `@main` — bleeding edge; use on low-risk sub-repos first
- `@v1` — stable tag (TBD); use once tagged
- Always pin to a commit SHA or tag for production

## Updating rules

1. Edit rule files under `security/`
2. Open PR → merge to `main`
3. All consumer repos pick up the change on next CI run
4. For breaking changes, cut a new major tag (`v2`) and let repos opt in

## Cross-org access (one-time setup)

Because `10k-challenge/prospector` and `10k-challenge/product-catalog` call a
repo under `rich1edwards`, the `10k-challenge` org must allow it:

1. GitHub org `10k-challenge` → Settings → Actions → General
2. Under **"Access"** → allow repositories to call reusable workflows from
   `rich1edwards/gold-rush-shared`
3. Alternatively: make this repo public (contains no secrets)
