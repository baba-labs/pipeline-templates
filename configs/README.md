# Shared lint configuration

These are the defaults the [`lint.yml`](../.github/workflows/lint.yml) workflow applies
when a repository has no configuration of its own.

They are duplicated here as files so you can read them without reading the workflow, and
so a repository can adopt them explicitly:

```bash
cp configs/prettierrc.json          .prettierrc
cp configs/prettierignore           .prettierignore
cp configs/markdownlint-cli2.yaml   .markdownlint-cli2.yaml
```

**Copy them when you intend to diverge.** If you are happy with the defaults, do not copy
them — an uncopied default follows this repository, and a copied one silently stops.

The root [pre-commit configuration](../.pre-commit-config.yaml) uses these same files,
so local formatting and Markdown checks match the reusable workflow.

## Why these settings

**Prettier `printWidth: 100`** — wide enough that prose and TypeScript both read well,
narrow enough for a side-by-side diff.

**`CHANGELOG.md` is ignored by both tools.** release-please generates it. Formatting it
produces a diff on every release and eventually a merge conflict during one.

**markdownlint `MD013` (line length) off** — prose should wrap where it reads best, not
at a column. Enforcing it makes people write worse sentences.

**`MD033` (inline HTML) off** — used deliberately in documentation for anchors and
detail blocks.

**`MD024` `siblings_only`** — ADRs repeat `Context`, `Decision` and `Consequences` by
design, and a rule that fights the template is the wrong rule.
