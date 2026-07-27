## Trooth Compliance Scan v1

Run a Trooth compliance preflight on every push and pull request. The action posts your Terraform plan to the Trooth API, scores it against the frameworks you pick, and fails the build when findings cross the line you set.

Free for public repositories at the Bronze tier. Get a key at https://www.trooth.co.

### Use it

```yaml
- uses: troothllc/trooth-action@v1
  with:
    api-key: ${{ secrets.TROOTH_API_KEY }}
    plan-file: plan.json
    frameworks: soc2,iso27001,eu-ai-act,nist-ai-rmf,hipaa
    fail-on: critical
```

### What you get

- A pass or fail gate tied to the `fail-on` severity you choose.
- A composite Trust Score and the raw API verdict.
- A link to the full report on your Public Trust Profile.
- A PR comment and a job summary with finding counts by severity.
- Outputs for downstream steps: `score`, `status`, `verdict`, `report-url`, `findings-count`.

### Frameworks

SOC 2, ISO 27001, EU AI Act, NIST AI RMF, HIPAA. Weight them with the `frameworks` input, or leave it empty to use everything enabled on your tier.

### Before you run it

- Produce a plan file: `terraform plan -out=tfplan` then `terraform show -json tfplan > plan.json`.
- Add your key as `secrets.TROOTH_API_KEY` under Settings, Secrets and variables, Actions.
- License: Apache 2.0.

Trooth automates. Trooth never signs your claims for you.
