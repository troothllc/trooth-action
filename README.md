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
      - uses: troothllc/trooth-action@v1
        with:
          path: ./infra
```

No key, no account and no secret. `contents: read` for the checkout is the only permission it needs: the action writes the job summary through the runner's own file rather than through the API, so it also works on pull requests from forks.

## Inputs

| Input | Required | Default | Description |
|---|---|---|---|
| `path` | No | `.` | Directory to read, relative to the workspace. |
| `version` | No | `0.4.2` | Version of the `trooth` CLI to run from npm. Pinned on purpose. Bump it deliberately. |
| `fail-on-inline-credentials` | No | `false` | Opt in: fail the step when `lint` counts one or more inline credential literals. |
| `fail-if-nothing-read` | No | `false` | Opt in: fail the step when the directory yields no infrastructure declarations. |

## Outputs

| Output | Description |
|---|---|
| `digest` | `sha256:` and a SHA-256 over the report's `facts` object in canonical form. The timestamp, the path and the CLI version are not part of it. The same tree read by the same CLI version produces the same digest. |
| `declarations-read` | How many declaration files were read. |
| `inline-credential-literals` | How many inline credential literals were counted. A count: the literals are never printed. |
| `report` | Path to the JSON fact document, for `actions/upload-artifact` if you want to keep it. |

```yaml
- uses: troothllc/trooth-action@v1
  id: lint
  with:
    path: ./infra

- uses: actions/upload-artifact@v4
  with:
    name: trooth-lint
    path: ${{ steps.lint.outputs.report }}

- run: echo "Digest ${{ steps.lint.outputs.digest }}"
```

The digest is evidence that a given state was observed, without publishing the tree it came from. Record it with the build, or in your own records.

## When the step fails

Three cases, and only three.

1. The action could not do its job. The path does not exist, or the CLI could not be installed, did not start, or stopped before producing a report. Nothing was read, so the step fails and the reason is in the annotation.
2. `fail-if-nothing-read` is on and the directory yielded no declarations. That usually means the action is pointed at the wrong place.
3. `fail-on-inline-credentials` is on and the count is not zero. A credential literal in infrastructure code is unambiguous. The count is a pattern match, at most one per line, on a quoted value of eight or more characters assigned to a name such as `password`, `secret`, `token` or `api_key`, and it is only a count: the literals are not printed, so find them with your own tooling.

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

`.tf`, `.tf.json`, Kubernetes YAML (anything carrying both `apiVersion` and `kind`), `terraform show -json` plan files and Dockerfiles. It is a declaration reader, not a full HCL parser: it matches patterns in the text of each file rather than parsing it.

## What it deliberately does not do

It issues no verdict, no severity and no rating. It checks nothing against any named standard or regulation. It does not rate, rank or grade your repository, and it sends nothing about your repository anywhere: its only network use is installing the pinned CLI from npm, the step points the CLI at an unroutable address, and `lint` makes no requests in any case.

These facts come from your own run on your own runner: Trooth does not witness them and signs nothing here. What the facts mean is your decision, not Trooth's.

## How the CLI gets there

The `trooth` package is installed from npm at the pinned version into a directory under `RUNNER_TEMP` and run by path, not through `npx`. `npx` resolves a package name against the checked-out project before it asks the registry, which in a repository whose own `package.json` is named `trooth` runs a bin that was never linked and exits 127. A run in which the read never happened must not show as successful, so a run that could not install or start the CLI fails as the action's own fault, with the reason in the annotation.

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
