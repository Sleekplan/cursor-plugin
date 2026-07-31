# Changelog

All notable changes to the Sleekplan Cursor plugin are documented here.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and
this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] — 2026-08-01

Initial release.

### Added

- Sleekplan MCP server connection — feedback, comments, changelog, votes, users,
  surveys, tags and workspace configuration.
- Five skills:
  - `sleekplan-changelog-from-commits` — draft a changelog entry from a commit
    range, a pull request, or a release.
  - `sleekplan-feedback-to-plan` — turn a feedback post into an implementation
    plan grounded in the current codebase.
  - `sleekplan-ship-and-update-roadmap` — move posts along the public roadmap as
    work lands, and notify the people who voted.
  - `sleekplan-triage-feedback` — dedupe, categorise, tag and route incoming
    feedback.
  - `sleekplan-prioritize-feedback` — rank the backlog by momentum, customer
    segment and effort.
- `sleekplan-workspace` rule — workspace key lookup, roadmap semantics, and
  confirmation gates for anything customer-facing.
