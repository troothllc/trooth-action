# trooth-action

Run `trooth lint` in GitHub Actions. **Advisory by default: it reports, and it does not fail your build over anything it reads unless you ask it to.**

The action reads what the infrastructure in a directory **declares** and writes that to the job summary: how many declaration files, from which sources, which regions and zones, how many storage declarations and how many of those declare encryption, how many rules are open to any address, how many inline credential literals. Counts, resource type names and region strings, with the path it read, the CLI version, the time of the read and the digest, and nothing else. Never a file name, a line, a value or a snippet.

It is a wrapper around the [`trooth`](https://github.com/troothllc/trooth-cli) CLI's `lint` command, which is local and offline by construction. Beyond the job summary and step outputs of your own workflow run (anyone who can see the run can read the summary), nothing about your repository is transmitted anywhere, and nothing is sent to Trooth.

**There is no verdict.** `lint` reports what your infrastructure declares and does not judge it. Declaring public ingress is not a failing, because a load balancer is supposed to be public. So this action cannot fail your workflow over what it reads unless you opt in, explicitly, to one of two gates that are yours to choose.

## Quick start

```yaml
name: Trooth lint

on:
  push:
    branches: [main]
  pull_request:

permissions:
  contents: read

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: troothllc/trooth-action@<full commit SHA>  # v1
        with:
          path: ./infra
```

**Pin by commit.** Replace `<full commit SHA>` with the 40-character commit that `v1` points at when you adopt it (`git ls-remote https://github.com/troothllc/trooth-action refs/tags/v1`), and let Dependabot or Renovate propose updates you review. A tag can be moved to different code; a commit cannot. `@v1` works and is what the examples below abbreviate to, but it runs whatever `v1` points at on the day of each run.

No key, no account and no secret. `contents: read` for the checkout is the only permission it needs: the action writes the job summary through the runner's own file rather than through the API, so it also works on pull requests from forks.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `path` | No | `.` | Directory to read, relative to the workspace. |
| `version` | No | the release pinned in `action.yml` | Version of the `trooth` CLI to run from npm. One exact release such as `0.5.0`; a range, a dist-tag such as `latest`, a URL or a path is refused before anything is installed. Pinned on purpose: the default moves only when this action is updated. |
| `allow-incomplete` | No | `false` | Keep the step green when `lint` could not read every selected file: one was over the size limit, did not parse or could not be read, or the walk hit its file limit. The summary says the read was incomplete either way. |
| `fail-on-inline-credentials` | No | `false` | Opt in: fail the step when `lint` counts one or more inline credential literals. |
| `fail-if-nothing-read` | No | `false` | Opt in: fail the step when the directory yields no infrastructure declarations. |

## Outputs

| Output | Description |
|---|---|
| `facts-digest` | `sha256:` and a SHA-256 over the report's `facts` object in canonical form. An aggregate of the counts: two different trees with the same counts share it. It does not identify file contents, a repository, a commit or a deployment. |
| `digest` | The same value as `facts-digest`, under its earlier name. Kept for existing workflows. |
| `completeness` | `complete` or `incomplete`. |
| `declarations-read` | How many declaration files were read. |
| `inline-credential-literals` | How many inline credential literals were counted. A count: the literals are never printed. |
| `report` | Path to this step's own JSON fact document, for `actions/upload-artifact`. Every invocation writes into its own new directory under `RUNNER_TEMP`, so two steps in one job never overwrite each other's report. |

```yaml
- uses: troothllc/trooth-action@v1
  id: lint
  with:
    path: ./infra

- uses: actions/upload-artifact@v4
  with:
    name: trooth-lint
    path: ${{ steps.lint.outputs.report }}

- run: echo "Facts digest ${{ steps.lint.outputs.facts-digest }}"
```

The facts digest tells you whether the counts changed between two runs of the same CLI version. It is not a hash of your files and does not identify a commit or a deployment, so do not record it as evidence of either.

## When the step fails

Four cases, and only four.

1. The action could not do its job. The path does not exist, or the CLI could not be installed, did not start, or stopped before producing a report. Nothing was read, so the step fails and the reason is in the annotation.
2. `fail-if-nothing-read` is on and the directory yielded no declarations. That usually means the action is pointed at the wrong place.
3. `fail-on-inline-credentials` is on and the count is not zero. A credential literal in infrastructure code is unambiguous. The count is one per parsed key named like `password`, `secret`, `token` or `api_key` that holds a literal string of eight or more characters, and it is only a count: the literals are not printed, so find them with your own tooling.
4. The read was incomplete and `allow-incomplete` is off. The counts describe part of the tree, and a green step would say otherwise.

Everything else is a summary a person reads. A successful run says the read happened. It does not say your infrastructure is good, and it is not evidence that Trooth has ingested anything.

## Opting in to a gate

```yaml
- uses: troothllc/trooth-action@v1
  with:
    path: ./infra
    fail-on-inline-credentials: "true"
    fail-if-nothing-read: "true"
```

## What it reads

`.tf`, `.tf.json`, Kubernetes YAML (anything carrying both `apiVersion` and `kind`), `terraform show -json` plan files and Dockerfiles. Since CLI 0.5.0 every file is parsed, and a file that does not parse is reported as invalid rather than read. Nothing is evaluated: a setting that depends on a variable, a local or a module is reported as unresolved, so a count can differ from what Terraform itself would plan. The job summary names the CLI version that did the read, and the [CLI's README](https://github.com/troothllc/trooth-cli) describes how each source is read.

## What it deliberately does not do

It issues no verdict, no severity and no rating. It checks nothing against any named standard or regulation. It does not rate, rank or grade your repository, and it sends nothing about your repository anywhere: its only network use is installing the pinned CLI from npm, the step points the CLI at an unroutable address, and `lint` makes no requests in any case.

These facts come from your own run on your own runner: Trooth does not witness them and signs nothing here. What the facts mean is your decision, not Trooth's.

## How the CLI gets there

The `trooth` package is installed from npm at one exact version, with install scripts disabled, into a directory under `RUNNER_TEMP`, and run by path, not through `npx`. npm checks the tarball against the registry's SHA-512 integrity value as it installs, and the action compares the installed package's own version with the one it asked for before running it. The CLI ships an `npm-shrinkwrap.json`, so its one dependency installs at the locked version too. `npx` resolves a package name against the checked-out project before it asks the registry, which in a repository whose own `package.json` is named `trooth` runs a bin that was never linked and exits 127. A run in which the read never happened must not show as successful, so a run that could not install or start the CLI fails as the action's own fault, with the reason in the annotation.

## Links

- The CLI: [`troothllc/trooth-cli`](https://github.com/troothllc/trooth-cli), and [trooth.co/cli](https://trooth.co/cli)
- The Trooth Network: [trooth.co/network](https://trooth.co/network)
- Developers: [trooth.co/developers](https://trooth.co/developers)
- Publish your own record, free: [trooth.co/get-started](https://trooth.co/get-started)
- Report a vulnerability: [Vulnerability Disclosure Policy](https://trooth.co/security/vulnerability-disclosure-policy)
- Contact: [trooth.co/contact](https://trooth.co/contact)

## License

Apache License 2.0. See [LICENSE](LICENSE).

Trooth signs what it witnessed. It never signs on a company's behalf.
