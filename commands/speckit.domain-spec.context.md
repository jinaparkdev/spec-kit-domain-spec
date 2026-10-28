---
description: "Read the domain specs relevant to the current feature before specifying or planning"
---

# Domain Spec Context

Load the canonical "current state" domain specs that the current feature touches,
so the feature spec and plan are written against today's rules instead of
against older per-feature specs. This command runs as a mandatory `before_specify`
and `before_plan` hook. It reads; it never writes.

## Hook Dispatch Contract

`/speckit-specify` and `/speckit-plan` read this mandatory hook from
`.specify/extensions.yml` and run `speckit.domain-spec.context` under the
invocation name your agent uses (for example `/speckit-domain-spec-context`,
or the `speckit-domain-spec-context` skill). Emit the dispatch line in the form
the core command asks for, for example:

```text
EXECUTE_COMMAND: speckit.domain-spec.context
```

## User Input

```text
$ARGUMENTS
```

When run as a `before_specify` hook, `$ARGUMENTS` is the feature description the
user gave to `/speckit-specify`. Use it to choose domains.

## Configuration

Read `.specify/extensions/domain-spec/domain-spec-config.yml`, then apply
`.specify/extensions/domain-spec/local-config.yml` on top if it exists. For any
missing key use the default documented in
`.specify/extensions/domain-spec/config-template.yml`. Below, `<domains_dir>`,
`<language>`, `labels.*` and the other names refer to these values.

If `<domains_dir>/index.md` does not exist, the project has no domain specs yet.
Say so in one line, suggest `speckit.domain-spec.bootstrap`, and stop without
blocking the calling command.

## Why

Feature directories (`<feature_dirs>`) are change history. A later feature often
overrides an earlier one without editing it, so an older feature spec can state
a rule that is no longer true. `<domains_dir>/` holds the current rules. When the
two disagree, `<domains_dir>/` wins.

## Steps

1. Read `<domains_dir>/index.md`. It lists every domain, its scope, its status
   (`labels.status_consolidated` or `labels.status_planned`) and the features
   that shaped it.
2. Decide which domains the feature touches:
   - Before specify: from `$ARGUMENTS`, plus a linked issue if one is given.
   - Before plan: from the feature's `spec.md`. If it has a
     `## <labels.affected_domains>` section, start from that list.
   - For each entry in `common_domains`, include `entry.path` when the feature
     matches `entry.include_when`.
3. For each chosen domain with status `labels.status_consolidated`, read
   `spec.md`, `data-model.md` (if present) and every file under `contracts/`.
   Skim `history.md` only to see which features already shaped the area.
4. For each chosen domain with status `labels.status_planned`, there is no
   canonical spec yet. Read the code and the feature specs listed in `index.md`,
   newest first, and treat the newest feature as authoritative where they
   conflict.
5. Read the `## <labels.needs_confirmation>` section of every chosen domain.
   Those rows record known gaps between the code and older specs or
   `secondary_docs`, and known suspected bugs. If the feature is about to rely
   on, change or fix one of them, say so explicitly.

## Output

Do not write files. Report to the caller in `<language>`:

- The chosen domains and why each was chosen.
- The current rules from those domains that the feature must respect or will
  change, each with its domain and rule ID (for example `loan` R-006).
- Any `labels.needs_confirmation` row the feature touches.
- Any older feature spec the user mentioned whose rule is no longer current, with
  the domain rule that replaced it.

When this runs before `/speckit-specify`, the feature spec must include a
`## <labels.affected_domains>` section listing the chosen domains, so the plan
and the update hook can start from it. Add the section even if the spec template
has no placeholder for it. When this runs before `/speckit-plan`, the plan's
Technical Context should cite the domain rules it depends on.
