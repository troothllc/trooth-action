# trooth-action

[![Release](https://img.shields.io/github/v/release/troothllc/trooth-action?label=release&color=D97706)](https://github.com/troothllc/trooth-action/releases)
[![Marketplace](https://img.shields.io/badge/marketplace-trooth--action-D97706)](https://github.com/marketplace/actions/trooth-compliance-scan)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

Witness your repo's security and AI posture on every push and keep your Trooth Network profile current. Scans against SOC 2, ISO 27001, the EU AI Act, NIST AI RMF, and HIPAA. Free for public repos.

## What it does

This GitHub Action calls the Trooth API on every push or pull request, scans against the frameworks you configure, and surfaces the results as:

| Result | What you get |
|---|---|
| Workflow status | Passes or fails based on the `fail-on` threshold |
| Standing | Your composite Trust Score |
| Report link | A link to the full scan report on your public trust record |
| PR comment | The scan summary, posted on the pull request (configurable) |

## Quick start

Add this workflow at `.github/workflows/trooth.yml`:

```yaml
name: Trooth Compliance

on:
  push:
    branches: [main]
  pull_request:

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Produce a Terraform plan in JSON for Trooth to evaluate.
      - uses: hashicorp/setup-terraform@v3
      - run: |
          terraform init
          terraform plan -out=tfplan
          terraform show -json tfplan > plan.json

      - uses: troothllc/trooth-action@v1
        with:
          api-key: ${{ secrets.TROOTH_API_KEY }}
          plan-file: plan.json
```

Get a free API key at [trooth.co](https://trooth.co) and add it to your repository as `secrets.TROOTH_API_KEY` (Settings, then Secrets and variables, then Actions).

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `api-key` | Yes | n/a | Your Trooth API key. Pass via `secrets.TROOTH_API_KEY`. |
| `plan-file` | Yes | n/a | Path to a `terraform show -json` plan file. |
| `frameworks` | No | All on your plan | Comma-separated: `soc2`, `iso27001`, `eu-ai-act`, `nist-ai-rmf`, `hipaa`. |
| `fail-on` | No | `critical` | Severity that fails the workflow: `critical`, `high`, `medium`, `low`, `none`. |
| `comment-on-pr` | No | `true` | Post the scan summary as a PR comment. |
| `trooth-host` | No | `https://api.trooth.co` | API host. Override for staging or self-hosted environments. |
| `github-token` | No | `${{ github.token }}` | Token used to post the PR comment. |

## Outputs

| Output | Description |
|---|---|
| `score` | Composite Trust Score. |
| `status` | `pass` or `fail` relative to the `fail-on` input. |
| `report-url` | URL of the full scan report. |
| `findings-count` | Total number of findings produced by the scan. |

Use outputs in subsequent steps:

```yaml
- uses: troothllc/trooth-action@v1
  id: trooth
  with:
    api-key: ${{ secrets.TROOTH_API_KEY }}

- name: Publish badge
  run: echo "Trust Score ${{ steps.trooth.outputs.score }}"
```

## Configuration examples

### Fail only on critical findings (default)

```yaml
- uses: troothllc/trooth-action@v1
  with:
    api-key: ${{ secrets.TROOTH_API_KEY }}
    fail-on: critical
```

### Scan only against the EU AI Act

```yaml
- uses: troothllc/trooth-action@v1
  with:
    api-key: ${{ secrets.TROOTH_API_KEY }}
    frameworks: eu-ai-act
```

### Report only, never fail

```yaml
- uses: troothllc/trooth-action@v1
  with:
    api-key: ${{ secrets.TROOTH_API_KEY }}
    fail-on: none
```

## Plans

Free for public repos. Paid plans add expanded framework coverage, Slack and Discord drift webhooks, and EU AI Act conformity attestation. See [trooth.co/pricing](https://trooth.co/pricing).

## Security

Pass your API key via GitHub Actions secrets. Never commit it to source. See [SECURITY.md](https://github.com/troothllc/.github/blob/main/SECURITY.md) for the vulnerability disclosure policy.

## License

Apache License 2.0. See [LICENSE](LICENSE).

## About Trooth

Trooth is the witnessed trust network for software and AI companies. A company gets witnessed once, across identity, security, privacy, and AI practices, each with a source and a date, and buyers and their AI agents read a current, signed record with no login. Get witnessed at [trooth.co/signup](https://trooth.co/signup).

[trooth.co](https://trooth.co) · [Security](https://trooth.co/security)
