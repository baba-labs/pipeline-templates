# pipeline-templates

Shared GitHub Actions workflows, actions and configuration for BaBa Labs repositories.

Consumers reference these rather than copying them, so a fix lands everywhere at once.

## Available workflows

| Workflow                                                     | Purpose                                                                                |
| ------------------------------------------------------------ | -------------------------------------------------------------------------------------- |
| [`lint.yml`](.github/workflows/lint.yml)                     | Prettier, markdownlint, Terraform fmt / validate / docs / tflint, Conventional Commits |
| [`pre-commit.yml`](.github/workflows/pre-commit.yml)         | Every hook in the repository's `.pre-commit-config.yaml`, over all files               |
| [`dotnet.yml`](.github/workflows/dotnet.yml)                 | .NET locked restore, build, `dotnet format` verification and tests                     |
| [`release-please.yml`](.github/workflows/release-please.yml) | Version bump, changelog and release PR; optionally moves the major tag                 |

## Using them

```yaml
# .github/workflows/pr.yml in the consuming repository
jobs:
  lint:
    uses: baba-labs/pipeline-templates/.github/workflows/lint.yml@v1
    permissions:
      contents: read
      pull-requests: read

  pre-commit:
    uses: baba-labs/pipeline-templates/.github/workflows/pre-commit.yml@v1

  dotnet:
    uses: baba-labs/pipeline-templates/.github/workflows/dotnet.yml@v1
    with:
      solution: MyApp.slnx
```

A worked example is in [`examples/pr.yml`](examples/pr.yml).

**Pin to `@v1`, never `@main`.** `v1` is a moving tag that follows the latest v1.x
release, so you get fixes without opting into breaking changes. Referencing `main` means
a commit here can break every repository's CI at once — which, for a small team, is the
difference between an inconvenience and losing a day.

`v1` is moved automatically: [`release.yml`](.github/workflows/release.yml) calls
`release-please.yml` with `update-major-tag: true`, so the tag follows each release in
the same run that creates it. Never move it by hand.

## Two things that trip people up

**Reusable workflows do not inherit permissions.** The calling job declares what the
called workflow needs. If a check that reads the pull request suddenly fails with a 403,
this is why.

**Private consuming repositories need access granted here.** Settings → Actions →
General → Access → _Accessible from repositories in the baba-labs organisation_.
Without it, the reference fails with a not-found error that says nothing about
permissions.

## Lint configuration

`lint.yml` uses the consuming repository's own Prettier and markdownlint config when one
exists, and the shared default otherwise. Either way it logs which it used — a check
that silently falls back is a check you wrongly believe is enforcing your standard.

The defaults, and the reasoning behind them, are in [`configs/`](configs/README.md).

Terraform checks skip themselves when the repository contains no `.tf` files. The run
summary states what ran and why, so a skip is visible rather than assumed.

## Pre-commit

`pre-commit.yml` runs `pre-commit run --all-files` with the repository's own
`.pre-commit-config.yaml`, so CI enforces exactly what contributors run locally. That
includes hooks `lint.yml` has no job for, such as secret detection and actionlint.

pre-commit is installed from `requirements-file` (default `requirements-dev.txt`) with
`--require-hashes --only-binary :all:`. **The file must be hash-locked**: every package,
including transitive ones, pinned with its hashes. Keep the top-level pins in
`requirements-dev.in` and generate the file with:

```bash
pip-compile --generate-hashes --allow-unsafe --strip-extras --no-emit-index-url \
  --output-file requirements-dev.txt requirements-dev.in
```

A plain `pre-commit==x.y.z` line is not enough: its dependencies would still resolve
afresh on every run, and a package published within range would be picked up silently.
Dependabot's `pip` ecosystem understands the `.in` / `.txt` pair.

Inputs: `python-version` (default `3.13`), `requirements-file`, `skip-hooks` (a
comma-separated list passed to pre-commit's `SKIP`, for hooks another job already
covers) and `runs-on` (default `ubuntu-latest`; call the workflow from a matrix to cover
several operating systems). Skipped hooks are listed in the run summary.

## .NET

`dotnet.yml` restores, builds, verifies formatting and runs tests for one solution or
project (`solution`, required). Warnings-as-errors, analyzers and code style belong in the
repository (`Directory.Build.props`, `.editorconfig`); the workflow enforces whatever the
repository declares. GitHub Actions sets `CI=true`, which a repository can use to switch on
`ContinuousIntegrationBuild`.

| Input                   | Default                 | What it does                                                       |
| ----------------------- | ----------------------- | ------------------------------------------------------------------ |
| `configuration`         | `Release`               | Build and test configuration                                       |
| `global-json-file`      | `global.json`           | Pins the SDK; also read to detect the test runner                  |
| `locked-restore`        | `true`                  | `dotnet restore --locked-mode`; a stale `packages.lock.json` fails |
| `nuget-cache`           | `true`                  | Caches NuGet packages, keyed on `nuget-lock-files`                 |
| `nuget-lock-files`      | `**/packages.lock.json` | Lock files used as the cache key                                   |
| `verify-format`, `test` | `true`                  | `dotnet format --verify-no-changes`; `dotnet test`                 |
| `runs-on`               | `ubuntu-latest`         | Runner label; use a matrix in the caller for several systems       |

`locked-restore` and `nuget-cache` need lock files (`RestorePackagesWithLockFile`). Turn
both off for a repository without them.

The test runner is detected, not configured: when `global.json` sets `test.runner` to
`Microsoft.Testing.Platform` the workflow runs `dotnet test --solution …`, otherwise
`dotnet test …` (VSTest). The run summary records what it found and which steps ran.

## Conventional Commits

The default is to lint the **pull request title**, not the branch commits.

Under squash merge the PR title becomes the commit on `main`, and that is what
release-please parses to decide the version bump. Branch commits are discarded, so
linting them adds friction without protecting anything — and a branch of immaculate
commits can still land a squashed `Update stuff` that release-please ignores.

Set `commit-lint: true` for any repository using merge or rebase-merge, where individual
commits do land.

Also worth setting in each repository: Settings → General → Pull Requests → _Default to
pull request title_ for squash merge messages. Otherwise GitHub sometimes offers the
branch's commit list instead, and someone will accept it.

## Third-party actions are pinned to commit SHAs

A tag like `@v4` is mutable. If an action's maintainer is compromised, or the repository
changes hands, your workflow runs their code with your token on your next build.

Every third-party action here is pinned to a full commit SHA with the version in a
trailing comment. Dependabot updates them weekly and the comment keeps the diff readable.

For most companies this is excessive. For one selling engineering rigour, it is the
answer you want ready when a client's security questionnaire asks about supply chain.

## Contributing

Start with this repository's [contributing guide](CONTRIBUTING.md), which includes the
cross-platform pre-commit setup. Organisation-wide conventions remain in
[baba-labs/.github](https://github.com/baba-labs/.github/blob/main/CONTRIBUTING.md).

Changing a workflow here changes CI for every repository that pins the tag you release
it under. Breaking changes need a major version, and `!` or a `BREAKING CHANGE:` footer
so release-please produces one.
