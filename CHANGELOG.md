# Changelog

## 2026-04-01

### Added

- `skills/guidedtrack/gt-cli-guide.md` — Guide for the `gt` CLI tool (push, pull, create, build, program management, raw API requests)

### Fixed (from rigorous cross-facet testing)

- `SKILL.md` — Corrected Run vs Preview vs Debug description: Debug/Split generates test data (marked "test"), not no data
- `language-reference.md` — Clarified that `*min:` and `*max:` are only valid for `slider` type questions (not `number`)
- `website-guide.md` — Distinguished editor hamburger menu from dashboard card dropdown menu (different items); added URL patterns for History and Duplicate; noted Delete is dashboard-only
- `gtlint-guide.md` — Documented that `goto-needs-reset-in-events` rule is non-functional in v0.15.0 due to `*events` parsing issue; clarified `required-subkeywords` does not enforce `*save:` on questions
- `gt-cli-guide.md` — Documented `gt push --build` 404 bug workaround; noted `gt init` requires interactive terminal input with manual config alternative

## 2026-03-30

### Changed

- Restructured as a Claude Code plugin (`skills/guidedtrack/` + `.claude-plugin/plugin.json`)
- Removed all monorepo-specific paths and fallbacks (tools/gtlint, skills/spark-wave-skill)
- Added YAML frontmatter to SKILL.md
- Removed dev tooling files (eslint, prettier, pnpm-workspace, package.json)

## 2026-03-29

### Added

- Initial skill creation with full GuidedTrack language support
- `SKILL.md` — Entry point with trigger conditions, core workflow, and key language facts
- `src/language-reference.md` — Complete GT syntax reference (keywords, data types, operators, methods, question types, service calls, events)
- `src/best-practices.md` — Principled coding recommendations derived from GT's design characteristics and general PL best practices
- `src/gtlint-guide.md` — GTLint installation, configuration, CLI usage, inline directives, and rule reference
- `src/website-guide.md` — Comprehensive guide to guidedtrack.com browser interactions (editor, publishing, embedding, data collection, settings, collaborators, services, program-to-program calls)
- Project scaffolding mirroring spark-wave-skill structure (package.json, eslint, prettier, etc.)

### Updated (from GT docs review)

- `src/language-reference.md` — Added: `*component` with `*with:` local scoping, `*switch:` (navigate without return), `*settings` keyword, `*database:` keyword, `*trigger:` with `*send:` for data passing, `*startup` special event, `*answers:` 2D collections, URL query string parameters, `data::store()` for custom CSV columns, datetime/duration operations, corrected event handler documentation
- `src/custom-services-guide.md` — New file: Custom Services backend development with `guidedtrack-db` library (table operations, Mango queries, Operation object, route handlers, complete examples)
