# Contributing

Contributions are welcome. Keep changes focused, document user-visible behavior, and
open a pull request against `main`.

## Local quality gates

This repository uses [pre-commit](https://pre-commit.com/) rather than platform-specific
shell scripts. Hook tools run in isolated environments managed by pre-commit, so the
same configuration is used on macOS and Windows.

Create a virtual environment and install the pinned development dependency:

```bash
# macOS
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements-dev.txt
```

```powershell
# Windows PowerShell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements-dev.txt
```

Install both the pre-commit and commit-message hooks:

```bash
pre-commit install --install-hooks
```

Run the complete suite once before opening a pull request:

```bash
pre-commit run --all-files
```

The hooks enforce:

- safe repository basics, including conflict markers, valid YAML/JSON, file size limits,
  LF line endings, and private-key detection;
- broad secret detection;
- Prettier and Markdown formatting;
- GitHub Actions schema validation and actionlint;
- Conventional Commit messages.

Formatting hooks update files in place. Review and stage those changes, then commit
again. Use `pre-commit autoupdate` when intentionally updating hook versions.

## Commit messages

Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/):

```text
feat: add Terraform validation workflow
fix(release): preserve the moving major tag
docs: explain reusable workflow permissions
```

The type and subject must be lowercase, the subject must not end with a full stop, and
the complete header must not exceed 100 characters.

Git permits bypassing hooks with `--no-verify`, but that should only be used to recover
from a broken local toolchain. Pull-request checks remain authoritative.

## Pull requests

- Keep pull requests small enough to review confidently.
- Explain the motivation and any compatibility impact.
- Add or update examples and documentation with behavior changes.
- Use a Conventional Commit-formatted pull-request title because squash merges use that
  title as the commit on `main`.
- Do not commit generated credentials, private keys, local environment files, or
  dependency caches.
