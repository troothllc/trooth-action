# Changelog

All notable changes to `trooth-action` are recorded here.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project uses [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-08-02

First public release. Run a Trooth compliance preflight on every push and pull request, and gate the workflow on the result.

### Added

- Composite action that posts a `terraform show -json` plan to the Trooth preflight API (`POST /v1/preflight`) and gates the workflow on the composite Trust Score.
- Framework weighting for SOC 2, ISO 27001, EU AI Act, NIST AI RMF, and HIPAA, selectable through the `frameworks` input.
- `fail-on` severity gate with five levels: `critical` (default), `high`, `medium`, `low`, `none`.
- Five outputs for downstream steps: `score`, `status`, `verdict`, `report-url`, `findings-count`.
- Pull request comment with the scan summary, on by default and controlled by `comment-on-pr`.
- Job summary table written to the run on every scan.
- `trooth-host` input to point the action at staging or a self-hosted environment.
- Input validation that fails fast with a clear message when the API key or plan file is missing.
- Apache 2.0 license, `CODEOWNERS`, and a usage example under `examples/`.

### Notes

- Free for public repositories at the Bronze tier. Get a key at https://www.trooth.co.
- Trooth automates. Trooth never signs your claims for you.

[1.0.0]: https://github.com/troothllc/trooth-action/releases/tag/v1.0.0
