# Contributing

Contributions are welcome. Keep changes focused, document user-visible behavior, and
open a pull request against `main`.

## Local quality gates

This repository uses [pre-commit](https://pre-commit.com/) rather than platform-specific
shell scripts. Hook tools run in isolated environments managed by pre-commit, so the
same configuration is used on macOS and Windows.

Create a virtual environment and install the development tools. `requirements-dev.txt` is
hash-locked (every package pinned with its hashes), generated from `requirements-dev.in`:

```bash
# macOS
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --require-hashes --only-binary :all: -r requirements-dev.txt
```

```powershell
# Windows PowerShell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --require-hashes --only-binary :all: -r requirements-dev.txt
```

Install both the pre-commit and commit-message hooks:

```bash
pre-commit install --install-hooks
```

Run the file-oriented suite once before opening a pull request:

```bash
pre-commit run --all-files
```

This runs every hook staged as `pre-commit` — it does not include the commitlint hook,
which only runs at the `commit-msg` stage (see below), since there is no single commit
message to validate against every file in the repository.

The file-oriented hooks enforce:

- safe repository basics, including conflict markers, valid YAML/JSON, file size limits,
  LF line endings, and private-key detection;
- broad secret detection;
- Prettier and Markdown formatting;
- GitHub Actions schema validation and actionlint.

Formatting hooks update files in place. Review and stage those changes, then commit
again. Use `pre-commit autoupdate` when intentionally updating hook versions.

To change a development tool version, edit `requirements-dev.in` and regenerate the lock
file with the `pip-compile` command in its header. Dependabot does this weekly.

## Commit messages

Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/):

```text
feat: add Terraform validation workflow
fix(release): preserve the moving major tag
docs: explain reusable workflow permissions
```

The type must be one of `build`, `chore`, `ci`, `docs`, `feat`, `fix`, `perf`,
`refactor`, `revert`, `style` or `test`. The subject must not be sentence case, Start
Case, PascalCase or UPPERCASE (`Add x` and `ADD X` are rejected; `add x` and `add X` are
both fine), must not end with a full stop, and the complete header must not exceed 100
characters. See `commitlint.config.cjs` for the exact rules.

To check a commit message without making a commit, write it to a file and pass that
file's path (`--commit-msg-filename -` is not supported; it silently skips validation):

```bash
echo "feat: add example" > /tmp/msg.txt
pre-commit run commitlint --hook-stage commit-msg --commit-msg-filename /tmp/msg.txt
```

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
