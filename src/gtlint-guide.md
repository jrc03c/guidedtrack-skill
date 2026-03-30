# GTLint Integration Guide

GTLint is a linter and formatter for GuidedTrack, inspired by ESLint and Prettier. It catches undefined variables, invalid keywords, incorrect indentation, and many other issues — and it auto-formats code for consistency.

## When to Use GTLint

- **Always** when writing GT code to local `.gt` files. Lint and format after every write.
- **Not necessary** when writing GT code directly in the guidedtrack.com browser editor (GTLint can't reach the browser editor). However, it's worth recommending local-first editing to users who may not know about GTLint.

## Installation

GTLint is published as `@jrc03c/gtlint` on npm. There is also a local copy at `tools/gtlint` in this monorepo.

### Check if already installed

```bash
npx gtlint --version 2>/dev/null && echo "gtlint is available" || echo "gtlint not found"
```

### Install globally (preferred for general use)

```bash
npm install -g @jrc03c/gtlint
```

### Install per-project

```bash
npm install --save-dev @jrc03c/gtlint
```

### Use the local monorepo copy (fallback)

If npm install fails or is undesirable, use the local copy directly:

```bash
node /path/to/tools/gtlint/bin/gtlint.js lint program.gt
```

The local copy may need to be built first:

```bash
cd /path/to/tools/gtlint && npm install && npm run build
```

## Usage

### Linting

```bash
# Lint a specific file
npx gtlint lint program.gt

# Lint all .gt files in a directory
npx gtlint lint src/

# Lint with quiet mode (errors only, no warnings)
npx gtlint lint --quiet program.gt

# Lint with JSON output (useful for programmatic consumption)
npx gtlint lint --format json program.gt

# Lint with compact output (one line per issue)
npx gtlint lint --format compact program.gt
```

### Formatting

```bash
# Preview formatted output (prints to stdout, does not modify file)
npx gtlint format program.gt

# Format and write back to file
npx gtlint format --write program.gt

# Format all .gt files in a directory
npx gtlint format --write src/
```

### Recommended Workflow

After writing or modifying a `.gt` file:

```bash
# 1. Format the file
npx gtlint format --write program.gt

# 2. Lint the file
npx gtlint lint program.gt
```

Format first, then lint. Formatting fixes whitespace and spacing issues that would otherwise appear as lint warnings.

## Configuration

Create a `gtlint.config.js` file in the project root:

```javascript
export default {
  // Glob patterns to ignore
  ignore: [
    "**/node_modules/**",
    "**/dist/**",
  ],

  // Formatter settings
  format: {
    insertFinalNewline: true,
    spaceAfterComma: true,
    spaceAroundArrow: true,
    spaceAroundOperators: true,
    spaceInsideBraces: 0,
    spaceInsideBrackets: 0,
    spaceInsideParens: 0,
    trimTrailingWhitespace: true,
  },

  // Linter rule severities: "off" | "warn" | "error"
  lint: {
    correctIndentation: "error",
    gotoNeedsResetInEvents: "warn",
    indentStyle: "error",
    noDuplicateLabels: "error",
    noEmptyBlocks: "error",
    noInlineArgument: "error",
    noInvalidGoto: "error",
    noSingleQuotes: "error",
    noStrayColon: "error",
    noUnclosedBracket: "error",
    noUnclosedString: "error",
    noUndefinedVars: "error",
    noUnreachableCode: "warn",
    noUnusedLabels: "warn",
    noUnusedVars: "warn",
    purchaseSubkeywordConstraints: "error",
    requiredSubkeywords: "error",
    validKeyword: "error",
    validSubKeyword: "error",
    validSubkeywordValue: "error",
  },
}
```

**Note**: Config file rule names use camelCase. Inline directive rule names use kebab-case.

## Inline Directives

Use comment-based directives to control linting/formatting for specific lines or sections.

### Suppress lint warnings

```
-- @gtlint-disable-next-line no-unused-vars
>> temp_debug_value = some_expression

-- @gtlint-disable no-undefined-vars
-- ... section where variables come from external sources ...
-- @gtlint-enable no-undefined-vars
```

### Suppress formatting

```
-- @gtformat-disable
-- ... manually formatted section ...
-- @gtformat-enable
```

### Suppress both lint and format

```
-- @gt-disable-next-line
>> carefully_formatted = expression
```

### Declare cross-program variables

These directives tell the linter about variables that are defined or used outside the current program, preventing false positives:

```
-- Variables received from a parent program or URL query string
-- @from-parent: study_id, participant_id
-- @from-url: source, campaign

-- Variables that will be used by a child program
-- @to-child: user_email, user_name

-- Variables set by a child program
-- @from-child: signup_result

-- Variables that appear in CSV exports
-- @to-csv: participant_age, total_score
```

## Lint Rules Quick Reference

| Rule | Default | What it catches |
|------|---------|----------------|
| `no-undefined-vars` | error | Variables used but never assigned |
| `no-unused-vars` | warn | Variables assigned but never read |
| `valid-keyword` | error | Unrecognized `*keyword:` |
| `valid-sub-keyword` | error | Invalid sub-keyword for its parent |
| `valid-subkeyword-value` | error | Invalid value for a sub-keyword |
| `no-invalid-goto` | error | `*goto:` to nonexistent label |
| `no-duplicate-labels` | error | Multiple `*label:` with same name |
| `no-unused-labels` | warn | `*label:` never referenced |
| `no-unreachable-code` | warn | Code after `*goto:` or `*return` |
| `no-unclosed-string` | error | Unclosed `"string` |
| `no-unclosed-bracket` | error | Unclosed `[`, `{`, or `(` |
| `no-single-quotes` | error | `'string'` instead of `"string"` |
| `indent-style` | error | Spaces instead of tabs |
| `correct-indentation` | error | Wrong indentation level |
| `no-empty-blocks` | error | Keyword with no body |
| `no-stray-colon` | error | Stray `:` in expressions |
| `no-inline-argument` | error | Invalid inline argument |
| `required-subkeywords` | error | Missing required sub-keywords |
| `goto-needs-reset-in-events` | warn | `*goto:` without `*reset:` in events |
| `purchase-subkeyword-constraints` | error | Invalid purchase sub-keyword combos |

## Output Examples

**Default (stylish) output:**

```
program.gt
  5:1   error    Undefined variable: user_name       no-undefined-vars
  12:2  warning  Unused variable: temp_value          no-unused-vars

1 error, 1 warning
```

**Exit codes:** 0 = no errors, 1 = errors found (warnings alone don't cause non-zero exit).
