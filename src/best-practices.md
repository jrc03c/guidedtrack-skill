# GuidedTrack Best Practices

These recommendations are derived from GuidedTrack's specific design characteristics — particularly its global scoping, lack of `*else:`, and use as a tool for surveys, experiments, and interactive applications — combined with general programming best practices from languages with similar constraints.

## Variable Naming and Management

### Use descriptive, namespaced names

Since all variables are global (even across parent/child programs), treat the variable namespace like a shared resource. Use prefixes or descriptive compound names to avoid collisions and make intent clear.

```
-- Bad: generic names that will collide across programs
>> x = 0
>> flag = 1
>> data = []
>> temp = ""

-- Good: descriptive names scoped by purpose
>> survey_response_count = 0
>> consent_given = 1
>> participant_scores = []
>> email_validation_result = ""
```

### Establish naming conventions

Pick a convention and stick to it. `snake_case` is the most natural fit for GT because:
- GT identifiers are case-sensitive, so `userName` and `username` are different variables — snake_case avoids this ambiguity.
- The GT documentation and built-in methods use lowercase names.

```
-- Recommended: snake_case throughout
>> participant_age = 0
>> total_score = 0
>> has_completed_intake = 0

-- Boolean-like variables: prefix with has_, is_, did_, should_
>> is_eligible = 0
>> has_consented = 0
>> did_complete_survey = 0
```

### Initialize variables explicitly

GT variables spring into existence on first assignment. Always initialize variables near the top of the program (or the top of the logical section where they're used) so readers can see what state the program manages.

```
-- Top of program: declare all state
>> participant_name = ""
>> participant_age = 0
>> responses = []
>> is_eligible = 0

-- ... rest of program uses these variables
```

### Clean up variables you pass to child programs

When using `*program:` to call child programs, be intentional about which variables you're passing. Document the interface with GTLint directives:

```
-- @to-child: participant_email, signup_source
>> participant_email = email_response
>> signup_source = "main_survey"
*program: @myteam/email-signup

-- After the child program returns, document what it set:
-- @from-child: signup_success, signup_error_message
```

## Control Flow

### Simulate if/else with complementary conditions

Since GT has no `*else:`, always write the complementary condition explicitly. Keep the two branches adjacent so they read as a logical unit.

```
-- Clear if/else pattern
*if: age >= 18
	You are eligible to participate.
*if: age < 18
	Sorry, you must be 18 or older.
```

For more complex branching, use a "result variable" pattern to separate the decision from the action:

```
-- Complex branching: decide first, act second
>> eligibility_status = "unknown"

*if: age >= 18 and has_consented = 1
	>> eligibility_status = "eligible"
*if: age < 18
	>> eligibility_status = "too_young"
*if: age >= 18 and has_consented = 0
	>> eligibility_status = "no_consent"

-- Now act on the result
*if: eligibility_status = "eligible"
	*goto: BeginSurvey
*if: eligibility_status = "too_young"
	We're sorry, you must be 18 or older.
	*quit
*if: eligibility_status = "no_consent"
	You must provide consent to continue.
	*goto: ConsentPage
```

### Use *label: and *goto: for page-level navigation, not logic

`*goto:` is a powerful but unstructured control flow tool. Reserve it for navigating between major sections of a program (like pages in a survey). Avoid using it for fine-grained logic — use `*if:` conditions instead.

```
-- Good: labels for major sections
*label: Introduction
	Welcome to our study.
	*button: Begin

*label: Demographics
	*question: What is your age?
		*type: number
		*save: participant_age
	*button: Continue

*label: ThankYou
	Thank you for participating!

-- Bad: goto as logic control
*label: CheckAge
	*if: participant_age < 18
		*goto: TooYoung
	*goto: OldEnough
```

### Prefer *program: over *goto: for reusable sections

`*program:` returns to the calling point (like a function call). `*goto:` does not return. If you have a reusable section of logic, put it in a separate program and call it with `*program:`.

## Program Structure

### Use a consistent section order

A well-structured GT program follows a predictable order:

```
-- 1. GTLint directives (variable declarations for cross-program communication)
-- @from-parent: study_id, participant_id
-- @to-parent: completion_status

-- 2. Variable initialization
>> response_count = 0
>> is_complete = 0

-- 3. Settings (if any)
*settings
	*name: My Survey

-- 4. Main content (pages, questions, logic)
*page
	Welcome to the survey.
	*button: Start

-- ... questions and logic ...

-- 5. Completion / cleanup
>> is_complete = 1
Thank you for completing the survey.
```

### Keep programs focused

Each GT program should do one thing. If a program is growing past ~200 lines, consider splitting it into child programs called with `*program:`. This also helps manage the global variable space — each program's "interface" is the set of variables it reads from and writes to, which you can document with `@from-parent:` and `@to-parent:` directives.

### Use *page to group multiple elements on one screen

Each `*question` already creates its own page automatically — you do **not** need to wrap individual questions in `*page`. The purpose of `*page` is to group multiple elements (questions, text, buttons) onto a single screen.

```
-- Bad: redundant *page around a single question
*page
	*question: What is your age?
		*type: number
		*save: participant_age
	*button: Next

-- Good: question gets its own page automatically
*question: What is your age?
	*type: number
	*save: participant_age

-- Good: *page groups multiple questions on one screen
*page
	*question: What is your first name?
		*type: text
		*save: first_name
	*question: What is your last name?
		*type: text
		*save: last_name
	*button: Next
```

## Questions

### Always use *save:

Every `*question:` should have a `*save:` sub-keyword. Without it, the response is lost.

```
-- Bad: response is discarded
*question: What is your name?
	*type: text

-- Good: response is stored
*question: What is your name?
	*type: text
	*save: participant_name
```

### Validate input at the point of collection

GT questions are required by default (use `*blank` to make them optional). Use `*min:` and `*max:` for numeric range validation.

```
*question: How old are you?
	*type: number
	*save: participant_age
	*min: 0
	*max: 150
```

### Use *tip: for clarification

```
*question: How would you rate your anxiety right now?
	*type: slider
	*save: anxiety_rating
	*min: 0
	*max: 100
	*tip: 0 = no anxiety at all, 100 = the most anxiety you've ever felt
```

## Service Calls

### Always handle both *success and *error

```
*service: MyAPI
	*path: /submit
	*method: POST
	*send: {"data" -> response_data}
	*success
		>> submission_id = it["id"]
	*error
		>> api_error = it
		There was a problem submitting your response. Please try again.
```

### Store API responses in descriptive variables

Don't leave response data in `it` — immediately assign it to a named variable in the `*success` block.

## HTML Blocks

### Minimize *html usage

Prefer native GT keywords over `*html` when possible. GT's built-in rendering handles responsive layout, accessibility, and consistent styling. Use `*html` only when you need something GT can't express natively (custom CSS, complex layouts, embedded widgets).

### Keep *html blocks small and focused

When you must use `*html`, keep blocks short and focused on a single visual element. Long `*html` blocks are hard to maintain and break GT's readability.

## Events

### Use *reset: before *goto: in event handlers

Event handlers run asynchronously. Using `*goto:` inside an event handler without `*reset:` can cause unexpected behavior. Always precede it with `*reset:`:

```
*events
	time_expired
		*reset
		*goto: TimeUp
```

## Comments

### Comment the "why," not the "what"

GT's keyword syntax is already quite readable. Comments should explain intent, decisions, or non-obvious behavior — not restate what the code does.

```
-- Bad: restates the code
-- Set the age to 0
>> participant_age = 0

-- Good: explains why
-- Default age to 0; will be overwritten by the intake question.
-- If the participant skips intake (returning users), we use 0 as a sentinel.
>> participant_age = 0
```

### Use comments to mark section boundaries in longer programs

```
-- ============================================
-- SECTION: Demographics
-- ============================================

*page
	*question: What is your age?
		...
```

## Error-Prone Patterns to Avoid

### Don't rely on variable existence for logic

Since GT variables are global and uninitialized variables cause runtime errors, always initialize variables before using them in conditions.

```
-- Bad: participant_age might not exist yet
*if: participant_age >= 18
	...

-- Good: initialized at top of program
>> participant_age = 0
...
*if: participant_age >= 18
	...
```

### Don't reuse variable names for different purposes

With global scope, a variable name should have one meaning throughout the program.

```
-- Bad: 'result' means different things at different points
>> result = api_response["status"]
...
>> result = score_1 + score_2

-- Good: distinct names for distinct purposes
>> api_status = api_response["status"]
...
>> total_score = score_1 + score_2
```

### Watch for off-by-one errors with 1-indexed collections

```
>> items = ["a", "b", "c"]
>> first = items[1]   -- "a" (correct)
>> last = items[items.size]   -- "c" (correct)

-- Common mistake from 0-indexed languages:
>> wrong = items[0]   -- ERROR or unexpected behavior
```
