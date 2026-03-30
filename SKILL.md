# GuidedTrack Skill

You are now operating as a GuidedTrack expert. GuidedTrack (GT) is a domain-specific language for building interactive web applications, surveys, experiments, and forms, developed by Spark Wave. Programs are written in `.gt` files (or directly in the browser at guidedtrack.com) and run as web applications.

## When This Skill Applies

- The user asks you to write, edit, or debug GuidedTrack code
- The user asks questions about the GT language (syntax, keywords, features)
- The user asks you to lint or format GT code
- The user asks about GuidedTrack as a product or platform
- You encounter `.gt` files in the project

## Reference Materials

Read these files from `./src` as needed:

- **`language-reference.md`** — Complete GT syntax, keywords, data types, operators, built-in methods. Read this before writing any GT code.
- **`best-practices.md`** — Principled coding recommendations for GT. Read this before writing non-trivial GT programs.
- **`gtlint-guide.md`** — How to install, configure, and run GTLint. Read this before linting or formatting.
- **`website-guide.md`** — How to interact with guidedtrack.com: creating programs, using the editor, publishing, embedding, managing data, configuring settings, and program-to-program calls. Read this before performing any browser-based GT tasks.
- **`custom-services-guide.md`** — How to build Custom Services (server-side backends) for GT programs, including the `guidedtrack-db` library, route handlers, and CouchDB Mango queries. Read this when the user needs a backend/database for their GT program.

For broader context about Spark Wave (the company behind GuidedTrack), see the `spark-wave-skill` in `skills/spark-wave-skill`. Reading it is optional and only relevant if the user asks about Spark Wave, Positly, Clearer Thinking, or other related products.

For the authoritative GT language specification, see `tools/gtlint/LANGUAGE_SPEC.md`. For the official function and keyword API reference, see `tools/gtlint/gt.pdf`.

## Core Workflow

When asked to write GT code:

1. **Read `src/language-reference.md`** to ensure you have the syntax details fresh.
2. **Read `src/best-practices.md`** for coding guidelines.
3. **Write the GT code** to a `.gt` file (or present it inline if the user asks for code in conversation).
4. **If you wrote a `.gt` file, lint and format it** (see below).

## Linting and Formatting

**When writing GT code to local `.gt` files**, always lint and format after writing:

```bash
# Format first (fixes spacing, whitespace)
npx gtlint format --write <file>

# Then lint (catches logical and structural errors)
npx gtlint lint <file>
```

If `gtlint` is not available, install it:

```bash
npm install -g @jrc03c/gtlint
```

If npm install fails, use the local monorepo copy:

```bash
node tools/gtlint/bin/gtlint.js lint <file>
```

**When writing GT code directly in the guidedtrack.com browser editor**, linting is not necessary. However, if the user is unaware of GTLint, mention that local editing with GTLint support is available (including a VSCode extension) and can catch errors before they reach the browser.

Fix any lint errors before presenting the code as complete. Warnings are informational — review them but they don't block completion.

## Key Language Facts

Keep these in mind when writing GT code:

- **Indentation**: Tabs only, never spaces. Indentation defines block structure.
- **No `*else:`**: Use complementary `*if:` conditions instead.
- **No `==`**: Use `=` for both assignment (in `>>` lines) and equality (in `*if:` conditions).
- **No `true`/`false` literals**: Use `1`/`0`, `*set:`, or `"true".decode("JSON")`.
- **1-indexed collections**: `items[1]` is the first element, not `items[0]`.
- **Global scope**: All variables are global within a program and shared across parent/child programs.
- **Strings**: Double quotes only. Single quotes are an error.
- **`*program:` returns; `*goto:` does not**: `*program:` is like a function call. `*goto:` is a one-way jump.

## Writing Good GT Code

Do not cargo-cult patterns from sample programs. Instead, derive your approach from:

1. The language reference (syntax and semantics)
2. The best practices guide (principled recommendations)
3. General programming knowledge (how other languages handle global scope, naming, modularity)

Specifically:

- Use descriptive `snake_case` variable names with meaningful prefixes (e.g., `is_eligible`, `survey_response_count`)
- Initialize all variables explicitly near the top of the program
- Document cross-program variable contracts with GTLint directives (`@from-parent:`, `@to-child:`, etc.)
- Keep programs focused; split into child programs when a program exceeds ~200 lines
- Use `*page` to control screen layout in user-facing programs
- Always include `*save:` on questions
- Handle both `*success` and `*error` on service calls

## Browser Workflows (guidedtrack.com)

When the user asks you to work with GT programs on guidedtrack.com, read `src/website-guide.md` first.

### Writing Code in the Browser Editor

**Do not type code character by character into the Ace editor** — auto-indent will compound with explicit tabs, producing incorrect nesting. Instead, use JavaScript to set the editor content:

```javascript
const editor = ace.edit('ace-editor');
editor.setValue(`your GT code here`, -1);
```

Use `\t` for tab characters in the template literal. Save afterward with Ctrl-S.

### Key Browser Concepts

- **Run vs Preview**: Run (`/run`) generates real data. Preview (`/preview`) and Split/Debug (`/debug`) do not.
- **Two ID formats**: Programs have a numeric ID (used in edit/settings URLs) and a short alphanumeric ID (used in run/publish URLs).
- **Embedding requires a URL whitelist**: After pasting embed code, the embedding page's URL must be registered on the Publish page.
- **Services are configured in Settings, not code**: The `*service:` name in GT code must match a service configured in Settings > Services.
- **Program-to-program calls**: Use `*program: program-name` for your own programs (no username prefix needed). Use `*program: @username/program-name` for others' programs.
