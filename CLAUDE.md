# GuidedTrack Skill

This skill teaches AI agents how to write, edit, debug, and lint GuidedTrack (GT) programs. It is designed to be actionable: agents should be able to produce correct GT code, write it to `.gt` files, and lint/format it using GTLint.

## Structure

- `SKILL.md` — Entry point loaded when the skill triggers. Contains the core behavioral instructions.
- `src/language-reference.md` — Condensed GT language reference for use during code generation.
- `src/best-practices.md` — Principled GT coding recommendations.
- `src/gtlint-guide.md` — GTLint installation, configuration, and usage.
- `src/website-guide.md` — How to interact with guidedtrack.com (editor, publishing, data, settings, etc.).
- `src/custom-services-guide.md` — Custom Services backend development (guidedtrack-db library, routes, CouchDB).

## Updating

If you learn something about GuidedTrack that meaningfully updates or corrects the information in this skill, please update the relevant source files. Since these files are designed to be read by AI agents, maintain the existing formatting density and style.

## Related

- The **spark-wave-skill** (`skills/spark-wave-skill`) provides broader context about Spark Wave (the company behind GuidedTrack) and its other projects.
- The **GTLint tool** (`tools/gtlint`) is the linter/formatter referenced by this skill. Its `LANGUAGE_SPEC.md` and `CLAUDE.md` are authoritative references.
