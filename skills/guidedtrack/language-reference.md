# GuidedTrack Language Reference

GuidedTrack (GT) is a domain-specific language for building interactive web applications, surveys, experiments, and forms. Programs run in the browser via [guidedtrack.com](https://guidedtrack.com).

## File Extension

`.gt`

## Lexical Basics

- **Indentation**: Tabs only (never spaces). Indentation defines block structure, like Python.
- **Comments**: `-- This is a comment`
- **Keywords**: `*keyword: value` (prefixed with `*`, suffixed with `:`)
- **Sub-keywords**: Same syntax, indented under a parent keyword.
- **Expressions**: `>> variable = expression` (prefixed with `>>`)
- **Plain text**: Lines without a prefix are displayed to the user.

## Data Types

| Type | Literal syntax | `.type` value |
|------|---------------|---------------|
| Number | `42`, `3.14` | `"number"` |
| String | `"double quotes only"` | `"string"` |
| Collection (array) | `[1, 2, 3]` | `"collection"` |
| Association (dict) | `{"key" -> "val"}` | `"association"` |
| Datetime | `calendar::date`, `calendar::now` | `"datetime"` |
| Duration | `5.seconds`, `2.hours` | `"duration"` |
| Boolean | No literal; use `1`/`0`, `*set: flag`, or `"true".decode("JSON")` | — |

**Collections are 1-indexed**, not 0-indexed.

## Operators

| Category | Operators |
|----------|-----------|
| Arithmetic | `+`, `-`, `*`, `/`, `%` |
| Comparison | `=`, `<`, `>`, `<=`, `>=` |
| Logical | `and`, `or`, `not` |
| Membership | `in` |

`=` serves double duty: assignment in `>>` lines, equality comparison in `*if:` conditions. There is no `==`.

## String Interpolation

Curly braces embed expressions in text and strings:

```
Welcome, {name}! Your score is {score * 100}%.
>> message = "Hello, {name}"
```

## Text Formatting

In displayed text (not URLs, paths, or technical values):

- **Bold**: `*text*`
- **Italic**: `/text/`

## Variables and Scoping

All variables are **globally scoped** within a program. There is no block scope, function scope, or lexical scope. Variables set inside `*if:`, `*for:`, `*while:`, or any other block are visible everywhere in the program.

Variables are also shared across parent and child programs (via `*program:` calls), making them effectively global across the entire call chain.

## Control Flow

### Conditionals

There is **no `*else:` or `*elseif:`**. Use multiple `*if:` statements:

```
*if: age >= 18
	You are eligible.
*if: age < 18
	You are not eligible.
```

### Loops

```
*while: count < 10
	>> count = count + 1

*for: item in items
	Display {item}

*for: i, val in items
	Item {i}: {val}

*repeat: 5
	This runs 5 times.
```

### Labels and Goto

```
*label: SectionStart
	Content here.

*goto: SectionStart
```

`*goto:` jumps to a label. It does not return. Use sparingly.

### Return

`*return` exits the current program (or subprogram called via `*program:`).

## Questions

```
*question: What is your name?
	*type: text
	*save: user_name
```

### Question Types

| `*type:` value | Input |
|----------------|-------|
| `text` | Single-line text |
| `paragraph` | Multi-line text |
| `number` | Numeric input |
| `choice` | Multiple choice (default) |
| `checkbox` | Multiple selection |
| `slider` | Numeric slider |
| `calendar` | Date/time picker |
| `ranking` | Rank-order items |

### Multiple Choice with Logic

Answer options can have indented blocks that execute when selected:

```
*question: Do you consent?
	*save: consent
	Yes
		>> consented = 1
	No
		>> consented = 0
		Thank you for your time.
		*quit
```

### Common Sub-Keywords for Questions

`*save:`, `*type:`, `*default:`, `*placeholder:`, `*tip:`, `*min:`, `*max:`, `*blank`, `*shuffle`, `*other`, `*multiple`, `*confirm`, `*searchable`, `*throwaway`, `*countdown:`, `*tags:`, `*answers:`, `*before:`, `*after:`, `*classes:`, `*icon:`, `*image:`

**Notes**:
- Questions are required by default. Use `*blank` to make a question optional (skippable). There is no `*required` sub-keyword for questions.
- `*min:` and `*max:` are only valid for `slider` type questions. Using them on other question types (including `number`) causes a compilation error.

### Dynamic Answer Lists with `*answers:`

`*answers:` can accept a collection or a 2D collection (for display/value pairs):

```
-- Simple list
>> color_options = ["Red", "Blue", "Green"]
*question: Pick a color
	*answers: color_options
	*save: chosen_color

-- 2D collection: [["Display Text", recordedValue], ...]
>> scale = [["Strongly agree", 5], ["Agree", 4], ["Neutral", 3], ["Disagree", 2], ["Strongly disagree", 1]]
*question: I enjoy this activity.
	*answers: scale
	*save: enjoyment_score
```

With 2D collections, the user sees the display text but the saved value is the second element.

## Pages and Navigation

Each `*question` automatically creates its own page. `*page` is used to group multiple elements (questions, text, buttons) onto a **single** screen:

```
-- This question gets its own page automatically — no *page needed
*question: What is your age?
	*type: number
	*save: participant_age

-- Use *page to put multiple things on one screen
*page
	*question: First name?
		*type: text
		*save: first_name
	*question: Last name?
		*type: text
		*save: last_name
	*button: Next
```

`*button:` creates a clickable button that advances. `*navigation` controls the navigation bar. `*progress:` shows a progress bar.

## Programs (Subprogram Calls)

```
*program: @username/program-name
```

This calls another GT program, then **returns to the next line** (unlike `*goto:`). Think of it like a function call.

Variables are shared between parent and child programs.

## Service Calls (HTTP Requests)

```
*service: MyAPI
	*path: /users/{user_id}
	*method: POST
	*send: {"name" -> user_name, "email" -> user_email}
	*success
		>> response = it
	*error
		>> error_msg = it
```

`it` is a special variable that holds the response body in `*success` or error info in `*error`.

## Events and Triggers

```
*events
	*startup
		-- Runs every time the program loads (including page refresh)
		*goto: Initialize
			*reset
	timer_tick
		>> elapsed = elapsed + 1
		*goto: TimerUpdate
			*reset

*trigger: timer_tick
*wait
```

Events run **asynchronously**. When triggered, execution jumps to the event definition. After the event code runs, execution resumes from the line after the `*events` block — **not** from where the trigger was called. This means:

- **Always use `*goto:` with `*reset` inside event handlers** to control where execution resumes. Without this, events can cause infinite loops.
- `*startup` is a special event that runs on every program load, but resumes at the user's last position rather than after the `*events` block.

### Sending Data to Events

```
*trigger: showAlert
	*send: {"message" -> "Something happened!", "level" -> "warning"}

*events
	showAlert
		>> alert_message = it["message"]
		>> alert_level = it["level"]
		*goto: DisplayAlert
			*reset
```

Data sent via `*send:` is available in the `it` variable inside the event handler (scoped to that block).

### Triggering Events from JavaScript (Embedded Programs)

When a GT program is embedded in a web page, events can be triggered from JavaScript:

```javascript
$(window).trigger("myEvent", { key: "value" })
```

## Switch (Program Navigation Without Return)

```
*switch: other-program
	*reset
```

`*switch:` navigates to another program **without returning** to the caller. This is different from `*program:` which returns. Without `*reset`, the target program resumes from its last saved state. With `*reset`, it starts from the beginning.

```
*question: Which lesson?
	Lesson 1
		*switch: Lesson 1
			*reset
	Lesson 2
		*switch: Lesson 2
			*reset
```

## Component (UI Containers with Click Handlers)

```
*component
	*classes: alert-info
	This is a styled content box.
	*click
		>> selected = 1
		*goto: HandleSelection
```

`*component` creates a bordered content box. Sub-keywords:
- `*classes:` — CSS/Bootstrap class names (e.g., `alert-info`, `alert-warning`)
- `*click` — Code to run when user clicks the component

### Local Scoping with `*with:`

`*with:` provides **local variable scoping** for click handlers inside loops — the only form of non-global scoping in GT:

```
*label: PeopleList
*for: person in people
	*component
		*header: {person["name"]}
		*with: person
		*click
			Name: {it["name"]}
			Age: {it["age"]}
			*button: Back
			*goto: PeopleList
```

Without `*with:`, all components would reference the same (last) value of `person` due to global scoping. `*with:` captures the current value, accessible as `it` inside the click handler.

## Settings

```
*settings
	*back: yes
	*menu: no
```

Controls program-wide UI:
- `*back:` — Enable/disable the back arrow button (`yes`/`no`)
- `*menu:` — Show/hide the run menu (`yes`/`no`)

## Database

```
*database: request
	*what: email
	*success
		>> user_email = it["email"]
	*error
		>> error_reason = it["reason"]
```

Retrieves user data from the GuidedTrack database. Currently only `*what: email` is supported. The `it` variable in `*success`/`*error` blocks is scoped to those blocks.

## URL Query String Parameters

Variables can be injected into a program via URL query parameters:

```
https://www.guidedtrack.com/programs/{short_id}/run?name=Alice&age=25
```

- Values are always strings. Convert to numbers with: `>> age = age * 1`
- Multiple values for the same key create a collection: `?colors=Red&colors=Blue`
- For collections with duplicates, use pipe-delimited values: `?grades=98|89|90|89` then `>> grades = grades.split("|")`

Use GTLint's `@from-url:` directive to suppress undefined-variable warnings for URL parameters.

## Custom CSV Columns with data::store

```
>> data::store("column_name", value)
```

Creates a named column in the CSV export with the given value. Useful when variable names are dynamic (e.g., inside loops):

```
*for: movie in movies
	*question: Rate {movie}?
		*answers: [5, 4, 3, 2, 1]
		*save: rating
	>> data::store(movie, rating)
```

This creates a separate CSV column for each movie title, rather than overwriting a single `rating` column.

## Other Keywords

| Keyword | Purpose |
|---------|---------|
| `*header:` | Section heading |
| `*image:` | Display an image (URL) |
| `*audio:` | Play audio (URL) |
| `*video:` | Embed video (URL) |
| `*html` | Embed raw HTML block |
| `*chart:` | Display a chart |
| `*email:` | Send an email |
| `*set:` | Set a variable to true |
| `*clear:` | Clear the screen |
| `*quit` | End the program |
| `*wait:` | Pause for a duration |
| `*experiment:` | A/B testing |
| `*randomize` | Randomize child block order |
| `*group:` | Named group for conditional display |
| `*login:` | User authentication |
| `*purchase:` | In-app purchases |
| `*share:` | Social sharing |
| `*summary` | Display a summary of responses |

## Built-in Methods

### String

| Method | Returns |
|--------|---------|
| `.size` | Character count |
| `.uppercase` | Uppercased copy |
| `.lowercase` | Lowercased copy |
| `.clean` | Trimmed/cleaned copy |
| `.find("x")` | Index of first occurrence (1-based), or 0 |
| `.count("x")` | Number of occurrences |
| `.split("x")` | Collection, split on delimiter |
| `.encode("url")` | URL-encoded string |
| `.decode("JSON")` | Parsed JSON value |

### Number

| Method | Returns |
|--------|---------|
| `.round` | Rounded to nearest integer |
| `.round(n)` | Rounded to n decimal places |
| `.seconds`, `.minutes`, `.hours`, `.days`, `.weeks`, `.months`, `.years` | Duration |

### Collection

| Method | Effect / Returns |
|--------|-----------------|
| `.size` | Element count |
| `.add(x)` | Appends element |
| `.insert(x, i)` | Inserts at position i |
| `.remove(i)` | Removes element at position i |
| `.erase(x)` | Removes first occurrence of value x |
| `.find(x)` | Index of x (1-based), or 0 |
| `.count(x)` | Number of occurrences of x |
| `.combine(other)` | Concatenates two collections |
| `.sort("asc")` / `.sort("desc")` | Sorts in place |
| `.shuffle` | Randomizes order |
| `.unique` | Removes duplicates |
| `.min`, `.max`, `.mean`, `.median` | Aggregate stats |

### Association

| Method | Effect / Returns |
|--------|-----------------|
| `.keys` | Collection of keys |
| `.remove("key")` | Removes a key |
| `.erase(value)` | Removes first key with that value |
| `.encode("JSON")` | JSON string |

### Any Variable

| Method | Returns |
|--------|---------|
| `.type` | Type name as string (e.g., `"string"`, `"number"`) |
| `.text` | String representation |

### Datetime and Duration

```
>> today = calendar::date
>> now = calendar::now
>> current_time = calendar::time

-- Create specific dates/times
>> christmas = calendar::date({"year" -> 2026, "month" -> 12, "day" -> 25})
>> noon = calendar::time({"hour" -> 12, "minute" -> 0})
>> next_wednesday = calendar::date({"weekday" -> "Wednesday"})

-- Arithmetic with durations
>> deadline = calendar::now + 7.days
>> elapsed = calendar::date - start_date
>> days_elapsed = elapsed.to("days")
```

Duration `.to()` accepts: `"seconds"`, `"minutes"`, `"hours"`, `"days"`, `"weeks"`, `"months"`, `"years"`.

## Things GT Does NOT Have

- `*else:` or `*elseif:` — use multiple `*if:` statements
- `==` — use `=` for both assignment and equality
- `true` / `false` literals — use `1`/`0`, `*set:`, or `.decode("JSON")`
- 0-indexed arrays — collections start at index 1
- Block/function scope — all variables are global (exception: `*component` with `*with:` provides local scoping for click handlers)
- Try/catch — use `*success`/`*error` blocks on service calls
