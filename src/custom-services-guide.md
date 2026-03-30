# GuidedTrack Custom Services Guide

Custom Services allow GT programs to have their own server-side backend with a database, written in JavaScript. They are configured and coded entirely within guidedtrack.com (Settings > Custom Services tab on the dashboard).

## Overview

A Custom Service consists of **routes** — JavaScript functions that handle HTTP requests from `*service:` calls in GT programs. Each route has an HTTP method (GET, POST, PUT, PATCH, DELETE) and a path. Routes can read/write data using the `guidedtrack-db` library, which provides access to a CouchDB-backed database.

## The `guidedtrack-db` Library

Every route handler imports the `guidedtrack` object:

```javascript
import guidedtrack from "guidedtrack-db";
```

### Table Access

```javascript
const table = guidedtrack.table("tableName");
```

Returns a `Table` object. Table names are arbitrary — tables are created on first use.

### Table Methods

#### `Table.insert(data)`

Inserts a record. Auto-generates `_id` and `created_at` fields.

```javascript
export const handler = async (event) => {
    var data_sent = JSON.parse(event.body);
    return await guidedtrack.table("participants").insert(data_sent).response();
};
```

#### `Table.find(id)`

Retrieves a single record by ID.

```javascript
export const handler = async (event) => {
    var id = event.queryStringParameters.id;
    return await guidedtrack.table("participants").find(id).response();
};
```

#### `Table.search(selector)`

Queries records using CouchDB Mango selectors.

```javascript
export const handler = async (event) => {
    var query = JSON.parse(event.body);
    return await guidedtrack.table("participants").search(query).response();
};
```

#### `Table.count()`

Returns the total number of records.

```javascript
const count = await guidedtrack.table("registrations").count();
```

### Mango Query Selectors

Selectors follow CouchDB Mango syntax:

```javascript
// Exact match
{ "name": "John Doe" }

// Regex
{ "name": { "$regex": ".* Doe" } }

// Empty selector = all records
{}
```

Full selector docs: https://docs.couchdb.org/en/stable/ddocs/mango.html

### The Operation Object

`Table.insert()`, `Table.find()`, and `Table.search()` return an `Operation` object with these methods:

#### Execution Methods

| Method | Returns | Best for |
|--------|---------|----------|
| `.response()` | AWS Lambda response (`{statusCode, body}`) | Route handlers (returns to GT `*service:` call) |
| `.values()` | Array of documents | `search()` results |
| `.value()` | Single document | `find()` results |

#### Query Modifiers (for `search()` only)

| Method | Effect |
|--------|--------|
| `.columns("col1", "col2")` | Return only specified fields |
| `.skip(n)` | Skip first n records (pagination) |
| `.limit(n)` | Return at most n records |
| `.sort({"field": "asc"})` | Sort results (accepts multiple sort objects) |
| `.average("column")` | Calculate average of a numeric column |

Methods are chainable:

```javascript
const results = await guidedtrack.table("scores")
    .search({"type": "professional"})
    .sort({"created_at": "desc"})
    .limit(250)
    .values();

const avg = await guidedtrack.table("scores")
    .search({})
    .limit(250)
    .sort({"created_at": "desc"})
    .average("percentile");
```

## Route Handler Structure

Every route handler must:
1. Be an `async` function named `handler`
2. Accept an `event` parameter
3. Return a response object with `statusCode` and `body`
4. Complete within **10 seconds**

```javascript
import guidedtrack from "guidedtrack-db";

export const handler = async (event) => {
    // Parse request body (for POST/PUT/PATCH)
    var data_sent = JSON.parse(event.body);

    // Access query parameters (for GET/DELETE)
    var id = event.queryStringParameters.id;

    // Return response
    return {
        statusCode: 200,
        body: JSON.stringify({ result: "success" })
    };
};
```

## Calling Custom Services from GT

Custom Services are called like any external service via `*service:`. The service name must match the Custom Service name configured on the dashboard.

```
*service: My Custom DB
	*method: POST
	*path: /participants
	*send: {"name" -> participant_name, "email" -> participant_email}
	*success
		>> participant_id = it["id"]
	*error
		Something went wrong: {it}
```

## Complete Examples

### Example 1: Registration with Quota Enforcement

**Route: POST /registrations**
```javascript
import guidedtrack from "guidedtrack-db";

export const handler = async (event) => {
    const max_registrations = 30;
    const count = await guidedtrack.table("registrations").count();

    if (count >= max_registrations)
        return {
            statusCode: 403,
            body: JSON.stringify({ error: "Registration is closed." })
        };

    var data_sent = JSON.parse(event.body);
    return await guidedtrack.table("registrations").insert(data_sent).response();
};
```

**Route: GET /registrations/count**
```javascript
import guidedtrack from "guidedtrack-db";

export const handler = async (event) => {
    const count = await guidedtrack.table("registrations").count();
    return { statusCode: 200, body: count };
};
```

**GT Program:**
```
>> max_registrations = 30

*service: Workshop Registration
	*path: /registrations/count
	*method: GET
	*success
		>> registration_count = it
	*error
		An error occurred: {it}

*if: registration_count >= max_registrations
	Sorry, registration is closed.
	*quit

*question: Your name?
	*type: text
	*save: participant_name

*service: Workshop Registration
	*path: /registrations
	*method: POST
	*send: {"name" -> participant_name}
	*success
		You're registered! Your ID: {it["id"]}
	*error
		Registration failed: {it}
```

### Example 2: Comparing User Results Against Averages

**Route: GET /scores/average**
```javascript
import guidedtrack from "guidedtrack-db";

export const handler = async (event) => {
    const results = guidedtrack.table("scores")
        .search({})
        .limit(250)
        .sort({"created_at": "desc"});

    const avg_percentile = await results.average("percentile");
    const avg_score = await results.average("score");

    return {
        statusCode: 200,
        body: JSON.stringify({
            percentile: avg_percentile,
            score: avg_score
        })
    };
};
```

### Example 3: Lookup by ID

**Route: GET /participants**
```javascript
import guidedtrack from "guidedtrack-db";

export const handler = async (event) => {
    var id = event.queryStringParameters.id;
    return await guidedtrack.table("participants").find(id).response();
};
```

**GT Program:**
```
*question: Enter your participant ID:
	*type: text
	*save: participant_id

*service: My Custom DB
	*method: GET
	*path: /participants?id={participant_id}
	*success
		Welcome back, {it["name"]}!
	*error
		ID not found.
```

## Auto-Generated Record Fields

When records are inserted, these fields are automatically added:
- `_id` — Unique identifier (auto-generated if not provided)
- `created_at` — Insertion timestamp
