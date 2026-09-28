# pipeline-templates

Shared GitHub Actions workflows, actions and configuration for BaBa Labs repositories.

Consumers reference these rather than copying them, so a fix lands everywhere at once.

## Available workflows

| Workflow                                                     | Purpose                                                                                |
| ------------------------------------------------------------ | -------------------------------------------------------------------------------------- |
| [`lint.yml`](.github/workflows/lint.yml)                     | Prettier, markdownlint, Terraform fmt / validate / docs / tflint, Conventional Commits |
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
