# [Domain name] Data Model

**As of**: [YYYY-MM-DD], commit [sha] on `main`

Describe the data this domain owns as it exists now: the schema after every
migration has run, or the keys of the files it reads and writes. Keep one of the
two section shapes below per store and delete the other.

## Stores

### Table `[schema.table]`

| Column | Type | Null | Default | Meaning |
|---|---|---|---|---|
| [column] | [type] | [N/Y] | [default] | [meaning] |

Constraints:

| Name | Kind | Definition |
|---|---|---|
| [constraint] | [PK / UK / FK / CHECK] | [definition] |

Indexes:

| Name | Columns |
|---|---|
| [index] | [columns] |

### File `[path/to/state.json]`

[Who writes it and who reads it, whether it is committed, whether each run
rewrites it or appends to it.]

| Key | Type | Meaning |
|---|---|---|
| [key] | [string / int / object / list] | [meaning] |

## Derived Values

Values computed on read, not stored.

| Value | How it is computed |
|---|---|
| [value] | [computation] |

## State Transitions

[Only for records that have a lifecycle: states, transition conditions, who
triggers each transition.]

## References

| From | To | How |
|---|---|---|
| [table.column / file.key] | [table.column / file.key] | [FK / same value / path] |
