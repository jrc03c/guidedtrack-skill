# GuidedTrack Website Guide

This guide covers how to interact with guidedtrack.com for creating, editing, publishing, and managing GT programs via the browser.

## Logging In

- Sign-in page: `https://www.guidedtrack.com/users/sign_in`
- Fields: Email address, Password
- After login, you land on the All Programs page (`/programs`)

## Programs Dashboard (`/programs`)

The main dashboard shows:
- **Programs** tab (with count) and **Custom Services** tab
- **"+ New program"** button and **"Hire an Expert"** button
- Search bar for filtering programs
- Each program card shows: name, created/updated dates, description, and action buttons (Edit, Preview, Data, Recruit Participants)
- Three-dot menu on each card for additional actions

## Creating a New Program

1. Click **"+ New program"** on the dashboard
2. Choose creation method: **Create with AI**, **Create from template**, or **Start from scratch**
3. Enter **Program name** and optional **Description**
4. Click **Save** to create the program and open the editor

## The Code Editor (`/programs/{id}/edit`)

### Layout

- **Top bar**: Program name (editable via pencil icon), Code/Split/Preview toggle, Explainer video link
- **Search bar**: Ctrl-K to search for functionality
- **Left sidebar**: Four category buttons for keyword helpers:
  - **Basics** (blue A): text, question, button, list, image, video, audio, chart
  - **Action** (green checkmark): navigation, login, settings, summary, email, share, trigger, service
  - **Display** (purple eye): points, progress, maintain, clear
  - **Flow** (purple arrows): program & switch, variable, if, set, encode & decode, label, goto, randomize
- **Ace editor**: Code editing area with line numbers, syntax highlighting, and code folding
- **Save button**: Bottom-right, also Ctrl-S

### Editor Modes

| Mode | URL suffix | Description |
|------|-----------|-------------|
| Code | `/edit` | Full-width code editor only |
| Split | `/debug` | Code on left, live preview on right. Shows green highlighting on currently executing lines. Shows tabs for parent/child programs. |
| Preview | `/preview` | Full-width live preview with device size toggles (Desktop, Tablet, Mobile, Freescale) and Toggle Design Tools |

### Writing Code via Browser Automation

**Important**: Do not type GT code directly into the Ace editor character by character — the editor's auto-indent will compound with explicit tab characters, producing incorrect nesting.

Instead, use JavaScript to set the editor content:

```javascript
const editor = ace.edit('ace-editor');
editor.setValue(`your GT code here`, -1);  // -1 moves cursor to start
```

Use `\t` for tab indentation in the template literal. After setting code, save with Ctrl-S or click the Save button.

### Saving

- **Ctrl-S** or click the **Save (Ctrl-S)** button
- Unsaved changes show a dot (●) in the browser tab title
- The dot disappears after successful save

## Program Navigation Menu

Click the **hamburger menu** (≡) in the top-right, or use the top nav bar on non-edit pages:

| Menu item | URL pattern | Purpose |
|-----------|------------|---------|
| Edit | `/programs/{id}/edit` | Code editor |
| Publish | `/programs/{short_id}/publish` | Distribution links and embed codes |
| Data | `/programs/{id}/summary` | Response data and analytics |
| Settings | `/programs/{id}/settings/access` | Program configuration |
| Recruit Participants | — | Links to Positly for participant recruitment |
| History | — | Version history |
| Run | `/programs/{short_id}/run` | Run the program live (generates real data) |
| Duplicate | — | Copy the program |
| Download code | — | Download the `.gt` source file |
| Delete | — | Delete the program |

**Note**: Programs have two ID formats:
- **Numeric ID** (e.g., `36778`): Used in edit, settings, and data URLs
- **Short alphanumeric ID** (e.g., `6k6cc08`): Used in run, publish, preview, and debug URLs

## Publishing (`/programs/{short_id}/publish`)

### Distribution Options

1. **Program link**: Direct URL for people to run the program (generates real data)
2. **Preview link**: URL for testing (does not generate real data)
3. **Website link**: HTML `<a>` tag to place on a website
4. **Embed code**: Full HTML embed with head and body sections

### Embedding a Program

The embed requires two HTML sections:

**Head section** (scripts and styles):
- jQuery (skip if already loaded)
- Bootstrap JS (skip if already loaded)
- `gt_interpreter.js` (the GT runtime)
- Bootstrap CSS (skip if already loaded)
- `guidedtrack.css`
- `<meta name="referrer" content="strict-origin-when-cross-origin">`

**Body section**:
- A `<div>` with `class="guidedtrack program_container"`, `id="{short_id}"`, and `data-environment="production"`
- Contains nested divs for navigation, points, maintain, status, and main content areas

**URL whitelist**: After the embed code, you must enter the full URLs of pages where the program will be embedded. The embedded program will not run if this step is skipped.

## Data (`/programs/{id}/summary`)

### Summary Tab

- Stats: Runs count, Finished percentage, Time to finish
- **"Download CSV"** button
- **"Analyze with Hypothesize"** button (links to Spark Wave's statistics tool)
- Links to question results and all runs

### Questions Tab (`/programs/{id}/questions`)

- Lists all questions with IDs and types
- Expandable cards showing response distributions (bar charts for multiple choice, response lists for text)
- Search bar for filtering questions

### Runs Tab (`/programs/{id}/runs`)

- Table with columns: Time Started (UTC), Time Finished (UTC), Minutes Spent, Type, Run ID, User
- **Type toggle**: Each run can be toggled between "data" and "test"
  - "data" runs count as real responses
  - "test" runs are excluded from analysis
  - Preview/debug runs are marked as "test" by default
  - Runs via the direct program link or `/run` URL are marked as "data"
- **Columns** dropdown to customize visible columns
- **Time filter** dropdown for date range
- **"Download CSV"** button

## Settings (`/programs/{id}/settings/...`)

Settings has multiple sub-tabs:

| Tab | Key fields |
|-----|-----------|
| **Access** | Access level (Restricted, Normal, Public Code); Collaborators list with "+ Add collaborators" |
| **Branding** | Public name, Display name for emails, Email Reply-To address, Logo URL, "Powered by GuidedTrack" toggle |
| **Appearance** | Visual customization |
| **Login** | User authentication settings |
| **Versioning** | Version management |
| **Run menu** | Run-time menu configuration |
| **Services** | External API services (see below) |
| **Social apps** | Social media integrations |
| **Mobile apps** | Mobile app settings |
| **Purchases** | In-app purchase configuration |
| **Privacy** | Privacy and data settings |
| **About** | Program metadata |

### Access Levels

- **Restricted**: Only collaborators can run or view the program
- **Normal** (default): Anyone with the link can run; only collaborators can see/edit code
- **Public Code**: Anyone can run, see, and copy the code

### Adding Collaborators

1. Go to Settings > Access
2. Click **"+ Add collaborators"**
3. Enter the collaborator's email address
4. Choose permissions (view-only or edit)

### Adding External Services

1. Go to Settings > Services
2. Click **"+ Add external service"**
3. Fill in: **Name** (used as identifier in `*service:` calls), **URL** (base URL), **Username**, **Password**
4. Optionally add custom **Headers** via "+ Add header"
5. Click **Save**

The service Name becomes the identifier in GT code: `*service: MyServiceName`

## Program-to-Program Calls

### Syntax

For programs you own:
```
*program: program-name
```

For programs owned by another user:
```
*program: @username/program-name
```

### Behavior

- The child program executes and then returns to the next line in the parent
- All variables are shared — the child can read and modify parent variables
- In Split view, the editor shows tabs for both parent and child programs
- Child program code is highlighted alongside the parent during debugging

## Running Programs

### Run vs Preview vs Debug

| Mode | URL | Generates data? | Shows code? |
|------|-----|----------------|-------------|
| Run | `/programs/{short_id}/run` | Yes (marked "data") | No |
| Preview | `/programs/{short_id}/preview` | No | No (but has device toggles) |
| Debug/Split | `/programs/{short_id}/debug` | Yes (marked "test") | Yes (side by side) |

### What the user sees when running

- The program renders as a clean web page
- Questions appear one at a time (unless no `*page` is used)
- Multiple choice questions advance immediately on click (no separate Submit)
- Text/number questions show a Submit button
- "Powered by GuidedTrack" footer appears (unless disabled in Branding settings)
- A "run menu" button (three dots) appears at the top for navigation controls
