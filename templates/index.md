# Domain Spec Index

This directory holds the canonical **current-state** spec of each domain. Look
here for the rules, data model and contracts that hold today.

Feature spec directories (created by `/speckit-specify`) are change history.
Read them to learn why a rule was set, not as the source of the current rule.
When a feature spec and a domain spec disagree, the domain spec is right.

## Domains

Status: `consolidated` = domain spec written, `planned` = only feature specs so far.

| Directory | Scope | Status | Source features |
|---|---|---|---|
| [`example/`](example/spec.md) | [what it covers] | planned | [001-short-name, 004-short-name] |

Until a `planned` domain is consolidated, read its code and source feature
specs oldest first to learn its current rules.

## Domain Directory Layout

| File | Contents |
|---|---|
| `spec.md` | Current rules, edge cases, decisions, related domains, items that need confirmation |
| `data-model.md` | Data the domain owns: tables, files, keys, state transitions (only if it owns data) |
| `contracts/api.md` | HTTP endpoints, parameters, responses, error meanings (only if it has an API) |
| `contracts/cli.md` | Commands, flags, environment variables, exit codes (only if it has a CLI) |
| `contracts/ui.md` | Screen behaviour rules (only if it has screens) |
| `history.md` | Features folded in, one line each, and retired rules |

`_common/` holds rules that several domains share. A domain spec links to a
common rule instead of restating it.

Templates live in [`_templates/`](_templates/).

## Authoring Rules

1. **The current code and its data are the source of truth.** Read the code
   and the persisted data first. Feature specs supply intent, reasons and edge
   cases.
2. **Write only what is true now.** "Previously X" goes only in `history.md`.
3. **Tag every rule with the feature that set it**, for example
   `(003-loan-limits)`. Rules that predate every feature spec have no tag.
   Changes made without a feature spec cite a commit or code path instead.
4. **When a document and the code disagree, write the code.** Record the
   difference under "Needs Confirmation" in `spec.md`, naming the document item
   and the code location.
5. **Leave out**: user story priorities, one-off migration procedures, lists of
   class or function names, results of a particular run, and the contents of
   tasks/research/quickstart files.
6. Keep paths, identifiers, field names, flags, environment variables, issue
   keys and error codes verbatim.

## File Name Constraints

These keep Spec Kit's scripts from mistaking domain specs for feature specs.

- Do not create files named `plan.md`, `tasks.md` or `README.md` under this
  directory.
- Do not name a domain directory like a feature directory (`001-...`, or an
  issue key such as `PROJ-123-...`).
