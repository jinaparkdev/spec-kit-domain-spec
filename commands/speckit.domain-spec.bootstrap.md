---
description: "Scaffold the domain spec directory and consolidate existing feature specs into domain specs"
---

# Domain Spec Bootstrap

Create the domain spec directory for a project that already has feature specs,
and write the first canonical spec for each domain from the current code and
the feature history. Run it once when adopting the extension, and again with a
domain name whenever a `planned` domain is ready to be consolidated. This
command is never run as a hook.

## User Input

```text
$ARGUMENTS
```

- Empty: scaffold if needed, then propose a domain map and wait for approval.
- `--scaffold-only`: create the directory, index and templates, then stop.
- One or more domain names (for example `loan _common/auth`): consolidate only
  those domains, which must already be listed in `index.md`.

## Configuration

Read `.specify/extensions/domain-spec/domain-spec-config.yml`, then apply
`.specify/extensions/domain-spec/local-config.yml` on top if it exists. For any
missing key use the default documented in
`.specify/extensions/domain-spec/config-template.yml`. Below, `<domains_dir>`,
`<language>`, `labels.*` and the other names refer to these values.

## Step 1: Scaffold

Skip anything that already exists; never overwrite.

1. Create `<domains_dir>/`.
2. Copy `.specify/extensions/domain-spec/templates/index.md` to
   `<domains_dir>/index.md`.
3. Copy `.specify/extensions/domain-spec/templates/domain/` to
   `<domains_dir>/_templates/`.
4. If `language` is not English, translate the prose of the new `index.md` and
   `_templates/` files into `<language>`. Translate headings only if
   `labels.*` were changed to match; otherwise keep the headings that
   `labels.*` name exactly as they are.

Stop here if `$ARGUMENTS` is `--scaffold-only`.

## Step 2: Propose a domain map (no domain names given)

1. List the feature directories (`feature_dirs`) oldest first. For each, read
   `spec.md` (title, user stories, Functional Requirements, Clarifications) and
   skim `data-model.md` and `contracts/` if present.
2. Read the code layout (`source_dirs`, or the repository) and the
   `data_sources` paths so domains follow the code's real boundaries, not only
   the feature titles.
3. Propose domains. A domain is an area with its own rules and data that
   several features changed or will change. Aim for a handful of domains, not
   one per feature. Put rules that several domains share (login and
   permissions, paging, ID formats, error bodies) into `_common/<name>`
   domains, and suggest a matching `common_domains` entry for each.
4. Present the map as a table: directory, scope, source features. Then list any
   feature that fits no domain, with why. **Wait for the user to approve or edit
   the map.** Do not write domain files before approval.
5. After approval, fill the domain table in `<domains_dir>/index.md` with every
   domain at status `labels.status_planned`, and print the `common_domains`
   block for the user to paste into the config. Do not edit the config file
   yourself.
6. Ask whether to consolidate all domains now or one at a time. Consolidate the
   chosen ones with Step 3.

## Step 3: Consolidate a domain

For each domain to consolidate:

1. Read every source feature listed for it in `index.md`, oldest first, and the
   code and data sources that implement it. Where features conflict, the newer
   one wins; where the code and any spec conflict, the code wins.
2. Copy the templates from `<domains_dir>/_templates/` into
   `<domains_dir>/<domain>/` and fill them in:
   - `spec.md`: only rules that are true in the current code. Number rules
     `R-001`, `R-002`, … within the domain. Tag each rule with its origin
     (`rule_tag`); a rule that predates every feature spec has no tag.
   - `data-model.md`: only if the domain owns persisted data from
     `data_sources`.
   - `contracts/`: one file per interface the domain exposes (`api.md`,
     `cli.md`, `ui.md`, `events.md`, …), only where it has one.
   - `history.md`: a first row for this consolidation with today's date and the
     current commit (`git rev-parse --short HEAD`), then one row per source
     feature. Rules that older specs state but the code no longer follows go
     into `## <labels.retired_rules>`.
   - Every gap between a spec, a `secondary_docs` entry and the code, and every
     intent the code cannot settle, goes into `## <labels.needs_confirmation>`.
3. Rules owned by a `common_domains` entry go only in that domain; link to it
   from the others.
4. Set the domain's status in `index.md` to `labels.status_consolidated`.
5. Follow the authoring rules and file-name constraints in `index.md`. Write
   prose in `<language>` and keep identifiers verbatim. Never invent a reason
   for a decision. Search exhaustively before claiming that something is
   unused (see `search_tool`).

Do not edit feature directories, `README.md`, agent instruction files,
`secondary_docs` or `protected_paths`.

## Output

Report in `<language>`: what was scaffolded, the approved domain map, the
domains consolidated (rule counts, data model and contracts written), the
`labels.needs_confirmation` rows recorded, and the domains still
`labels.status_planned`.
