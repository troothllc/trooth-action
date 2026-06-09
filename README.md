# trooth-action

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Trooth](https://img.shields.io/badge/built%20by-Trooth-00B563)](https://www.trooth.co)

Run a Trooth compliance scan on every push. SOC 2, ISO 27001, EU AI Act, NIST AI RMF, and HIPAA — continuous, automated, and free for public repos at the Bronze tier.

## What it does

This GitHub Action calls the Trooth API on every push or pull request, runs a compliance scan against the frameworks you configure, and surfaces the results as:

- A workflow status (passes or fails based on the `fail-on` threshold)
- A Trust Score (300 to 850)
- A link to the full scan report in your Trooth Public Trust Profile
- A PR comment with the scan summary (configurable)

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
      - uses: troothllc/trooth-action@v1
        with:
          api-key: ${{ secrets.TROOTH_API_KEY }}
```

Get a free API key at [trooth.co](https://www.trooth.co) and add it to your repository as `secrets.TROOTH_API_KEY` (Settings → Secrets and variables → Actions).

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `api-key` | Yes | — | Your Trooth API key. Pass via `secrets.TROOTH_API_KEY`. |
| `frameworks` | No | All on your tier | Comma-separated: `soc2`, `iso27001`, `eu-ai-act`, `nist-ai-rmf`, `hipaa`. |
| `fail-on` | No | `critical` | Severity that fails the workflow: `critical`, `high`, `medium`, `low`, `none`. |
| `comment-on-pr` | No | `true` | Post the scan summary as a PR comment. |
| `trooth-host` | No | `https://api.trooth.co` | API host. Override for staging or self-hosted environments. |

## Outputs

| Output | Description |
|---|---|
| `score` | Composite Trust Score (300 to 850). |
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
  run: echo "Trust Score - ${{ steps.trooth.outputs.score }}"
```

## Configuration examples

### Fail only on critical findings (default)

```yaml
- uses: troothllc/trooth-action@v1
  with:
    api-key: ${{ secrets.TROOTH_API_KEY }}
    fail-on: critical
```

### Scan only against EU AI Act

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

## Tier mapping

| Tier | Scan cadence | What this action does |
|---|---|---|
| Bronze (free) | On every push | Continuous multi-pillar scan with the standard framework coverage |
| Silver | On every push | Adds expanded framework coverage and Slack/Discord drift webhooks |
| Gold | On every push | Adds the full pillar engine, EU AI Act conformity workspace, and priority delivery |

See [trooth.co/pricing](https://www.trooth.co/pricing).

## Status during pre-launch

The Trooth API begins production scans on **August 2, 2026**, the EU AI Act Article 50 enforcement date. Before that date, this Action runs in scaffold mode — it accepts your inputs, validates configuration, and emits placeholder outputs so you can wire it into your workflow now and have full scans land automatically the day the API goes live.

## Security

Pass your API key via GitHub Actions secrets. Never commit it to source. See [SECURITY.md](https://github.com/troothllc/.github/blob/main/SECURITY.md) for the vulnerability disclosure policy.

## License

Apache License 2.0. See [LICENSE](LICENSE).

## About Trooth

Trooth provides cryptographic compliance infrastructure for AI products. Continuous monitoring against SOC 2, ISO 27001, EU AI Act, NIST AI RMF, and HIPAA. Free at Bronze.

[trooth.co](https://www.trooth.co) · [Trust Center](https://www.trooth.co/security)
