# pipeline-templates

Shared GitHub Actions workflows and lint configuration for BaBa Labs repositories. It is
**public** and meant to show how we work, so write for an outside reader, and never mention
unreleased products, clients or internal plans in files, commits or PRs.

## Layout

| Path                                      | What it is                                                                                                                      |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `.github/workflows/lint.yml`              | Reusable: Prettier, markdownlint, Terraform fmt/validate/docs/tflint, PR title (and optionally commits) as Conventional Commits |
| `.github/workflows/pre-commit.yml`        | Reusable: every hook in the caller's `.pre-commit-config.yaml`, installed from a hash-locked requirements file                  |
| `.github/workflows/dotnet.yml`            | Reusable: .NET locked restore, build, `dotnet format` verification and tests; test runner detected from `global.json`           |
| `.github/workflows/release-please.yml`    | Reusable: release PR and changelog via the release manager GitHub App; `update-major-tag` moves `vN` to each release            |
| `.github/workflows/pr.yml`, `release.yml` | This repository running its own workflows through local `./` references                                                         |
| `configs/`                                | Shared Prettier and markdownlint defaults, plus the proprietary licence template for private repositories                       |
| `examples/pr.yml`                         | What a consuming repository copies                                                                                              |
| `.local-only/`                            | Gitignored. Holds the GitHub App private key. **Never commit, print or move anything from it**                                  |

## The contract with consumers

Consumers reference `baba-labs/pipeline-templates/.github/workflows/<name>.yml@v1`. Every
change is a change to CI in every repository that pins the current major tag.

- **Inputs, outputs, secrets and job names are public API.** Removing or renaming one, or
  changing a default so existing callers behave differently, is a breaking change: use `!` or
  a `BREAKING CHANGE:` footer so release-please cuts a new major.
- Reusable workflows do not inherit permissions. If a job needs a new permission, callers
  must grant it — breaking unless it is documented and optional.
- `vars` and `secrets` in a called workflow resolve from the caller's repository or
  organisation. Document any new variable in the README.
- Never tag by hand. release-please creates `vX.Y.Z`; `release.yml` then moves `vX`.

## Rules for workflow changes

- Pin every third-party action to a full commit SHA with the version as a trailing comment
  (`uses: owner/action@<sha> # v1.2.3`). Resolve the SHA from the upstream tag; never guess.
- The shared defaults exist twice: as files in `configs/` and inline in `lint.yml` (the
  fallback written when a repository has no config). **Change both together.**
- Keep jobs skippable and loud about it: a check that skips must say why in the run summary.
- Least privilege: top-level `permissions: contents: read`, widened per job only as needed.
- Check out with `persist-credentials: false` unless the job pushes (only the release job does).
- Install Python tools from a hash-locked requirements file with `--require-hashes
--only-binary :all:`; `requirements-dev.txt` is generated from `requirements-dev.in`.
- Update `README.md` and `examples/` in the same PR as any behaviour change.

## Local checks

```bash
python3 -m venv .venv && source .venv/bin/activate
python -m pip install --require-hashes --only-binary :all: -r requirements-dev.txt
pre-commit install --install-hooks
pre-commit run --all-files
```

Hooks cover YAML/JSON validity, private-key and secret detection, Prettier, markdownlint,
GitHub workflow schema, actionlint, and commitlint on the commit message. Prettier and
markdownlint use `configs/` explicitly because the repository root has no config of its own.
A workflow change can only be proven on GitHub: the PR's own `PR` run exercises `lint.yml`,
and `release.yml` only runs after merge.
