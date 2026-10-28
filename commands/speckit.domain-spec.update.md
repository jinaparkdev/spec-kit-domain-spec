---
description: "Fold the implemented feature into the canonical domain specs"
---

# Domain Spec Update

After a feature is implemented, update the canonical domain specs under
`<domains_dir>/` so they describe the code as it now is. This command runs as a
mandatory `after_implement` hook. Run it manually too after a change that did
not go through `/speckit-implement` but changed behaviour, such as a bug fix
(pass the feature directory, an `extra_change_dirs` directory or a short
description as `$ARGUMENTS`).

## Hook Dispatch Contract

`/speckit-implement` reads this mandatory `after_implement` hook from
`.specify/extensions.yml` and runs `speckit.domain-spec.update` under the
invocation name your agent uses. Emit the dispatch line in the form the core
command asks for, for example:

```text
EXECUTE_COMMAND: speckit.domain-spec.update
```

If other `after_implement` hooks are registered (code review, auto-commit),
this one should run after review and before commit so the commit includes the
domain spec changes.

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Configuration

Read `.specify/extensions/domain-spec/domain-spec-config.yml`, then apply
`.specify/extensions/domain-spec/local-config.yml` on top if it exists. For any
missing key use the default documented in
`.specify/extensions/domain-spec/config-template.yml`. Below, `<domains_dir>`,
`<language>`, `labels.*` and the other names refer to these values.

If `<domains_dir>/index.md` does not exist, say so, suggest
`speckit.domain-spec.bootstrap`, and pass without edits.

## Feature Context Resolution

Resolve `FEATURE_DIR` from `.specify/feature.json` (`feature_directory`) first,
then from `.specify/scripts/bash/check-prerequisites.sh --json --paths-only`
(or `.specify/scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly`).
If `$ARGUMENTS` names a directory matching `feature_dirs` or `extra_change_dirs`,
use that instead.

## Inputs

- `<domains_dir>/index.md` — authoring rules and file-name constraints. Follow
  them exactly.
- `<FEATURE_DIR>/spec.md` — especially `## <labels.affected_domains>`,
  Functional Requirements and Clarifications. For an `extra_change_dirs`
  directory, read whatever documents it holds (assessment, fix notes).
- `<FEATURE_DIR>/data-model.md` and `<FEATURE_DIR>/contracts/*` if present.
- The implementation diff: merge-base with `origin/<base_branch>` (or
  `<base_branch>`) to `HEAD`, plus uncommitted changes. Without git, ask the
  user which files changed.
- Changes in the diff to any `data_sources` path: new, renamed or removed
  fields, keys, columns or tables, and changed meaning or value sets. These
  belong in the touched domain's `data-model.md`.

## Rules

1. **The code is the source of truth.** Write what the merged code does, not
   what the feature spec intended. Where they differ, write the code and add a
   row to that domain's `## <labels.needs_confirmation>` naming the feature spec
   item and the code location.
2. **Edit only domains the diff actually changed.** Start from
   `## <labels.affected_domains>`, then add any domain whose code, data source,
   endpoint, CLI flag or screen appears in the diff. Drop any listed domain the
   diff did not touch.
3. **Write only what is now true.** Replace superseded rules in place. Do not
   keep "previously X" in `spec.md`, `data-model.md` or `contracts/`. The old
   rule goes into the `## <labels.retired_rules>` table of `history.md` with the
   feature that replaced it.
4. **Tag every changed or new rule** with its origin. With
   `rule_tag: feature_dir` use the feature directory name, for example
   `(012-loan-renewal)`. With `rule_tag: issue_key` use the key matching
   `issue_key_pattern` from the directory name or spec, for example `(LIB-412)`.
5. **Add one row to `history.md`** of each touched domain: the tag from rule 4,
   a link to the feature's `spec.md` (or the change directory), and one line on
   what changed in that domain.
6. **Shared rules live in one place.** For each `common_domains` entry, rules
   described by `entry.owns` are written only in `<domains_dir>/<entry.path>/`.
   Other domains link to it and do not restate it.
7. **Resolve rows the feature fixed.** If the diff fixes a recorded
   `labels.needs_confirmation` row, delete it and record the fix in
   `history.md`.
8. **Never invent a reason.** In `## <labels.decisions>`, give a reason only if
   the feature spec, its Clarifications or a code comment states one.
9. **Do not touch** feature directories (`feature_dirs`), `extra_change_dirs`,
   `README.md`, agent instruction files (`AGENTS.md`, `CLAUDE.md`,
   `.github/copilot-instructions.md`, `.cursor/rules/`, and the like),
   `secondary_docs` or any `protected_paths` entry.
10. **Search exhaustively before claiming absence.** Before saying something has
    no caller, reader or writer, search `source_dirs` (or the whole repository)
    across all file types. With `search_tool: auto`, use graft if it is
    available, else `rg`, else `grep -rna` (plain `grep -r` silently skips files
    it thinks are binary).
11. **Do not write a partial canonical spec.** If a touched domain still has
    status `labels.status_planned` in `index.md`, add the feature to that
    domain's row in `index.md` and report that the domain still needs
    consolidation (`speckit.domain-spec.bootstrap <domain>`).
12. **Write prose in `<language>`.** Keep paths, identifiers, field names, flags,
    environment variables, issue keys and error codes verbatim.

## Output

- Edit the domain spec files in place.
- Update `<domains_dir>/index.md` if a domain's source-feature list changed.
- Report in `<language>`: the domains changed, the rules added/changed/retired
  with their IDs, the `labels.needs_confirmation` rows added or resolved, and
  any domain you expected to change but did not (with why).

## Gate Policy

- If the diff changes behaviour in a `labels.status_consolidated` domain and
  that domain's spec was not updated, report it as an Important issue and stop
  before completion.
- If no domain behaviour changed (for example a pure refactor or test-only
  change), say so and pass without edits.
