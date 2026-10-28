# Changelog

All notable changes to this extension are documented here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and versions follow
[Semantic Versioning](https://semver.org/).

## [0.1.0] - Unreleased

### Added

- `speckit.domain-spec.context`: loads the domain specs a feature touches
  before `/speckit-specify` and `/speckit-plan` (mandatory hooks).
- `speckit.domain-spec.update`: folds the implementation diff into the touched
  domain specs after `/speckit-implement` (mandatory hook), with a gate when a
  consolidated domain changed but its spec did not.
- `speckit.domain-spec.bootstrap`: scaffolds `specs/_domains/`, proposes a
  domain map from existing feature specs, and consolidates approved domains.
- Templates for `index.md` and per-domain `spec.md`, `data-model.md` and
  `history.md`.
- Configuration for language, directory layout, rule tags, shared-rule domains,
  data sources, protected paths, search tool and section labels.
