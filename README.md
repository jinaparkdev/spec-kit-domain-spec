# Domain Spec Maintenance for Spec Kit

A [Spec Kit](https://github.com/github/spec-kit) extension that keeps one
canonical, current-state spec per domain next to your feature specs, and keeps
it current automatically through hooks.

> **Feature specs are history. Domain specs are the source of truth.**

## The problem

`/speckit-specify` writes one directory per feature: `specs/001-…`,
`specs/002-…`, and so on. A later feature often changes a rule an earlier one
set, without editing the earlier spec. After a few dozen features, nobody can
tell which spec still describes the system, and the agent writing feature 040
happily builds on a rule that feature 017 retired.

## What this extension does

- **Organises specs by domain.** Each domain under `specs/_domains/<domain>/`
  has a fixed layout: `spec.md` (current rules, edge cases, decisions),
  `data-model.md`, `contracts/` and `history.md`. Rules several domains share
  (auth, paging, ID formats) live once under `_common/` and are linked, not
  copied.
- **Reads before writing, in both directions, on hooks.**
  - `before_specify` and `before_plan`: load the domain specs the feature
    touches, so the spec and plan are written against today's rules, and note
    the affected domains in the feature spec.
  - `after_implement`: fold what was actually implemented back into those
    domain specs, from the diff, with the code as the source of truth.
- **Keeps history out of the rules.** Superseded rules move to
  `history.md` → *Retired Rules*. Every rule carries a tag naming the feature
  that set it. Gaps between specs and code are recorded under
  *Needs Confirmation* instead of being guessed away.
- **Bootstraps from an existing project.** `speckit.domain-spec.bootstrap`
  proposes a domain map from your existing feature specs and code, waits for
  your approval, and writes the first canonical specs.

### How it compares

| | domain-spec | [archive](https://github.com/stn1slv/spec-kit-archive) |
|---|---|---|
| Unit | One spec per domain, plus shared `_common/` rules | One merged `spec.md` under `.specify/memory/` |
| When it runs | Automatically: before specify/plan, after implement | Manually, after merge |
| Feeds specify/plan | Yes, loads relevant domains first | No |
| Structure | `spec.md`, `data-model.md`, `contracts/`, `history.md` | Single file |

If you only want an end-of-project archive, `archive` is simpler. If you want
the agent to work from current rules on every feature, use this one. Other
related extensions: blueprint-index, memory-loader, docguard, arch-governance.

## Requirements

- Spec Kit `>=0.8.5`
- Optional: `git` (for the implementation diff), `rg` (faster, binary-safe
  search). The graft code index is used instead if installed.

Works with any agent Spec Kit supports (Claude Code, GitHub Copilot, Cursor,
Codex, Gemini, …). The commands are plain Markdown; Spec Kit registers them in
each agent's command or skill format.

## Installation

```bash
# From a release archive
specify extension add domain-spec --from https://github.com/jinaparkdev/spec-kit-domain-spec/archive/refs/tags/v0.1.0.zip

# From a local checkout
specify extension add --dev /path/to/spec-kit-domain-spec
```

The extension is listed in the Spec Kit community catalog, but that catalog
is discovery-only: `specify extension search domain-spec` finds it, and
`specify extension add domain-spec` without `--from` does not install it.
Use the release archive URL above.

Installing registers the three commands and hooks, and copies the config
template to `.specify/extensions/domain-spec/domain-spec-config.yml`.

After installing, check `.specify/extensions.yml`. If other extensions also
hook `after_implement` (code review, auto-commit), the installer may append
`speckit.domain-spec.update` after the commit hook. Move it so it runs after
review and before commit.

Then scaffold the domain directory:

```text
/speckit-domain-spec-bootstrap --scaffold-only
```

or, in a project that already has feature specs, run it without arguments to
get a proposed domain map.

### Adopting in a project that already keeps domain specs

If the project already has a `specs/_domains/` layout, for example from a
hand-written version of these commands, keep the specs and replace only the
commands:

1. Remove the old extension directory if it uses the same id
   (`git rm -r .specify/extensions/domain-spec`), then install as above.
2. Move project-specific rules from the old commands into
   `domain-spec-config.yml`: feature directory glob, bug-fix directories
   (`extra_change_dirs`), rule tag format, shared-rule domains
   (`common_domains`), data sources, documents that may lag the code
   (`secondary_docs`).
3. Set `labels` to the headings and status words your existing specs use, so
   the commands find them. Your feature spec template's affected-domains
   heading must match `labels.affected_domains`.
4. Do not run `bootstrap`; the specs are already consolidated. Run
   `/speckit-domain-spec-context <a feature description>` once and check that
   it picks the expected domains and does not write anything.
5. If you keep agent skills or commands in version control, check them after
   installing: the installer regenerates them for the active agent only and
   overwrites existing files with the same name.

## Commands

| Command | Hook | What it does |
|---|---|---|
| `speckit.domain-spec.context` | `before_specify`, `before_plan` | Reads `index.md`, picks the domains the feature touches, loads their rules and open questions, and reports them. Never writes. |
| `speckit.domain-spec.update` | `after_implement` | Updates the touched domain specs from the implementation diff, adds `history.md` rows, retires superseded rules. Stops with an Important issue if a consolidated domain changed but its spec did not. |
| `speckit.domain-spec.bootstrap` | – | Scaffolds `specs/_domains/`, proposes a domain map from existing feature specs, and consolidates domains you approve. |

Run `update` manually after changes that bypass `/speckit-implement`, such as
bug fixes: `/speckit-domain-spec-update specs/014-overdue-fees`.

## Configuration

Edit `.specify/extensions/domain-spec/domain-spec-config.yml`. Put personal
overrides in `local-config.yml` next to it (gitignored). Every key is optional.

| Key | Default | Purpose |
|---|---|---|
| `language` | `English` | Language of reports and spec prose. Identifiers stay verbatim. |
| `domains_dir` | `specs/_domains` | Where domain specs live. |
| `feature_dirs` | `specs/[0-9][0-9][0-9]-*` | Glob for feature spec directories, e.g. `specs/PROJ-*` for Jira-keyed branches. |
| `extra_change_dirs` | `[]` | Other change records `update` accepts, e.g. a bug-fix extension's directories. |
| `base_branch` | `main` | Diff base for `update`. |
| `rule_tag` | `feature_dir` | Tag rules with the feature directory name or an issue key (`issue_key`, matched by `issue_key_pattern`). |
| `common_domains` | `[]` | Shared-rule domains: when to load each (`include_when`) and which rules only it may hold (`owns`). |
| `data_sources` | `[]` | Globs for persisted data whose shape goes in `data-model.md` (migrations, JSON state…). |
| `source_dirs` | `[]` | Where the code lives; empty means the whole repository. |
| `secondary_docs` | `[]` | Documents that may lag the code; read for intent, disagreements recorded. |
| `protected_paths` | `[]` | Extra files `update` must never edit. |
| `search_tool` | `auto` | `auto` (graft → rg → `grep -rna`), or force `rg`, `graft`, `grep`. |
| `labels` | English headings | Section and status labels the commands look for, if your templates use another language. |

## Example

A fictional library lending app, *Bookshelf*, with sequential feature
branches.

```yaml
# .specify/extensions/domain-spec/domain-spec-config.yml
language: English
feature_dirs: "specs/[0-9][0-9][0-9]-*"
source_dirs: [api/, web/]
data_sources:
  - path: "api/migrations/*.sql"
    kind: "SQL migrations - the schema is the result of applying all of them"
common_domains:
  - path: _common/auth
    include_when: "the feature logs in, checks roles, or changes who may borrow"
    owns: "login, session lifetime, member and librarian roles"
  - path: _common/pagination
    include_when: "the feature lists records or adds sorting or filtering"
    owns: "page size, cursor format, sort and filter parameters"
```

```text
specs/
├── 001-catalog-search/        ← feature specs: history
├── 002-loan-checkout/
├── 007-loan-renewal/
└── _domains/                  ← domain specs: source of truth
    ├── index.md
    ├── _templates/
    ├── _common/
    │   ├── auth/spec.md
    │   └── pagination/spec.md
    ├── catalog/
    │   ├── spec.md
    │   ├── data-model.md
    │   ├── contracts/api.md
    │   └── history.md
    └── loan/
        ├── spec.md
        ├── data-model.md
        ├── contracts/api.md
        ├── contracts/ui.md
        └── history.md
```

A rule in `loan/spec.md` after feature 007:

```markdown
### Renewal

- **R-004**: A loan can be renewed at most twice. (007-loan-renewal)
- **R-005**: A loan with a pending hold by another member cannot be renewed. (007-loan-renewal)
```

and the matching `loan/history.md`:

```markdown
| Feature | Spec | What changed in this domain |
|---|---|---|
| Initial consolidation | (from code, 2026-03-02, 4f1c2ab) | Everything |
| 007-loan-renewal | [`specs/007-loan-renewal/`](../../007-loan-renewal/spec.md) | Renewal limit 1 → 2; holds block renewal |

## Retired Rules

| Old rule | Where it appears | Replaced by |
|---|---|---|
| A loan can be renewed once | 002-loan-checkout FR-009 | 007-loan-renewal |
```

When someone starts feature 012 with `/speckit-specify "let members renew
overdue loans"`, the `before_specify` hook loads `loan` (and `_common/auth`,
because borrowing rights are involved) and reports R-004 and R-005, so the new
spec is written against the current limit instead of the one in feature 002.

### Using another language

Set `language` and translate `labels` to match your templates. For Korean:

```yaml
language: Korean
labels:
  affected_domains: "영향 도메인"
  needs_confirmation: "확인 필요"
  retired_rules: "폐기된 규칙"
  decisions: "결정 사항"
  status_consolidated: "정리됨"
  status_planned: "예정"
```

`bootstrap` translates the scaffolded index and templates into `language`.

## Troubleshooting

- **The context hook says there are no domain specs.** Run
  `speckit.domain-spec.bootstrap`.
- **`update` stops with an Important issue.** A consolidated domain's behaviour
  changed but its spec was not updated. Re-run `update`, or update the spec by
  hand and re-run to confirm.
- **A domain is still `planned`.** `update` will not write a partial spec for it.
  Run `speckit.domain-spec.bootstrap <domain>`.
- **Hooks do not run.** Check `.specify/extensions.yml` lists the
  `domain-spec` hooks as enabled.
- **Reports come back in a different language than `language`.** Your agent's
  own instructions (for example a user-level `CLAUDE.md` or `AGENTS.md`) take
  precedence for chat output. Spec prose still follows `language`.

## Contributing

Issues and pull requests are welcome. Please describe the project layout you
tried it on (feature naming, data sources, agent) so changes stay general.

## License

[MIT](LICENSE)
