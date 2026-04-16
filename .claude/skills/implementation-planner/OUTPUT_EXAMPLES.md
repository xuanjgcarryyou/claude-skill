# Output Examples: Implementation Planner

Two full worked examples — one small system (3–4 components), one larger system (7 components).

---

## Example 1: Small System — Personal CLI Expense Tracker

### Input Spec (from idea-refiner output)

> **System Components:**
> - `cli.py`: Main entry point, parses subcommands
> - `db.py`: SQLite adapter (create, insert, query)
> - `export.py`: CSV export logic
> - `config.json`: Default categories and DB path
>
> **Tech Stack:** Python, SQLite, `click`, `csv` stdlib
>
> **API / Interface Outline:**
> ```
> expense add <amount> <category> [note]
> expense list [--month YYYY-MM] [--category <cat>]
> expense export [--month YYYY-MM] [--out <file>]
> expense categories
> ```

---

## Implementation Plan

### Architecture Summary
A personal CLI expense tracker that logs expenses with categories to a local SQLite database. Users interact via terminal commands: add expenses, list by month/category, and export to CSV. Single user, single machine, no network dependencies.

### Stack Confirmation
- Language: Python 3.10+
- CLI: `click`
- Database: SQLite via `sqlite3` stdlib
- Export: `csv` stdlib
- Config: plain `config.json` read with `json` stdlib

### Component Analysis

**cli.py**
- Chosen approach: `click` command group with subcommands (`add`, `list`, `export`, `categories`)
- Why: `click` handles argument parsing, type validation, and help text cleanly. One decorator per command, no manual `argparse` boilerplate.
- Key risks: Low. `click` is mature and well-understood. Risk is only in how options chain together (e.g., `--month` applies to both `list` and `export`).

**db.py**
- Chosen approach: Thin wrapper around `sqlite3` stdlib — one class `ExpenseDB` with methods for each operation. No ORM.
- Why: The schema is trivially simple (one table, ~5 columns). An ORM adds zero value here and would be a new dependency. Raw `sqlite3` is faster to write and easier to understand.
- Key risks: SQLite file path handling on different OS path separators — test on the target platform.

**export.py**
- Chosen approach: Single function `export_to_csv(rows, output_path)` using `csv.DictWriter`. Pulled from `db.py` results.
- Why: No library needed. `csv.DictWriter` handles quoting and encoding correctly for this use case.
- Key risks: None. CSV export is deterministic and well-covered by stdlib.

**config.json**
- Chosen approach: Static JSON file read at startup. Holds `db_path` and `categories` list. Loaded once in a `config.py` module that all others import.
- Why: No need for a config library. JSON is human-editable and version-controllable. A dedicated `config.py` module prevents every file from doing raw JSON parsing.
- Key risks: If the config file is missing or malformed, the error must be caught early and clearly reported. Do not let it fail silently.

### Dependency Map
```
cli.py → needs → db.py, export.py, config.py
db.py  → needs → config.py (for db_path)
export.py → needs → db.py (for row data)
config.py → independent (no dependencies)
```

### Build Phases
- **Phase 1 (Foundation):** config.py, db.py (schema + init)
- **Phase 2 (Core):** cli.py `add` command, cli.py `list` command, cli.py `categories` command
- **Phase 3 (Integration):** export.py + cli.py `export` command wired together
- **Phase 4 (Secondary):** Error message polish, help text, edge case handling

### Parallelization Opportunities
- Task 3 (`add` command) and Task 4 (`list` + `categories` commands) can run in parallel — both depend on Task 2 (db.py) but do not depend on each other.

---

### Task Packages

---

## Task 1: Config Module and DB Schema

**Phase:** 1
**Can run parallel with:** none
**Depends on:** none

**Objective:**
Create the `config.py` module and initialize the SQLite database with the expenses table schema.

**Scope:**
Included:
- `config.py` — loads and exposes `config.json` values
- `config.json` — initial file with default categories and db_path
- `db.py` — `ExpenseDB` class with `init_db()` method that creates the table if not exists

Not included:
- Query methods on `ExpenseDB` (those are Task 2)
- Any CLI code

**Input (what must exist before starting):**
- Python 3.10+ environment
- `click` installed

**Output (what this task produces):**
- `config.py` with `get_config()` returning `{ "db_path": str, "categories": list[str] }`
- `config.json` with defaults
- `db.py` with `ExpenseDB` class and `init_db()` that creates the `expenses` table

**Implementation Method:**
- Chosen approach: `config.py` loads `config.json` from the same directory using `pathlib.Path(__file__).parent / "config.json"`. `ExpenseDB.__init__` calls `init_db()` on every instantiation (CREATE TABLE IF NOT EXISTS — idempotent).
- Key patterns: Keep `config.py` as a thin reader. No global state — always return a fresh dict.

**Interface Contract:**
```python
# config.py
def get_config() -> dict:
    # Returns: { "db_path": str, "categories": list[str] }

# db.py
class ExpenseDB:
    def __init__(self, db_path: str): ...
    def init_db(self) -> None: ...  # Creates table if not exists
```

**Constraints:**
- `config.json` must be human-editable — no binary or escaped formats
- `init_db()` must be idempotent (safe to call multiple times)
- Config load failure must raise a clear `RuntimeError` with a readable message

**Context (for subagent):**
DB schema to implement:
```sql
CREATE TABLE IF NOT EXISTS expenses (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    amount REAL NOT NULL,
    category TEXT NOT NULL,
    note TEXT,
    created_at TEXT NOT NULL  -- ISO 8601 format: YYYY-MM-DD
);
```

Default `config.json`:
```json
{
  "db_path": "~/.expense-tracker/expenses.db",
  "categories": ["food", "transport", "subscriptions", "health", "utilities", "other"]
}
```

**Risks / Unknowns:**
- `db_path` may use `~` — must expand with `os.path.expanduser()` and create parent directory if missing
- No migration strategy needed for v1 — schema is fixed

---

## Task 2: ExpenseDB Query Methods

**Phase:** 1
**Can run parallel with:** none
**Depends on:** Task 1

**Objective:**
Implement all query methods on `ExpenseDB` that the CLI commands will call.

**Scope:**
Included:
- `insert_expense()` method
- `get_expenses()` method with optional month and category filters
- `get_categories()` method (reads from config, not DB)

Not included:
- Any CLI parsing
- CSV export logic

**Input (what must exist before starting):**
- Task 1 complete: `db.py` with `ExpenseDB` class and initialized schema

**Output (what this task produces):**
- Updated `db.py` with all query methods implemented and tested

**Implementation Method:**
- Chosen approach: Parameterized SQL queries using `sqlite3`'s `?` placeholders. No query builder.
- Key patterns: Return rows as `list[dict]` using `sqlite3.Row` with `row_factory = sqlite3.Row`. This makes downstream formatting and CSV export clean.

**Interface Contract:**
```python
class ExpenseDB:
    def insert_expense(self, amount: float, category: str, note: str | None) -> None: ...
    def get_expenses(self, month: str | None = None, category: str | None = None) -> list[dict]: ...
    # month format: "YYYY-MM" — filters by created_at LIKE "YYYY-MM-%"
    # Returns list of dicts: { id, amount, category, note, created_at }
```

**Constraints:**
- Must use parameterized queries — no string interpolation in SQL
- `get_expenses` must handle all filter combinations: no filter, month only, category only, both

**Context (for subagent):**
Schema from Task 1:
```sql
expenses(id, amount, category, note, created_at TEXT -- "YYYY-MM-DD")
```

**Risks / Unknowns:**
- Month filtering uses `LIKE "YYYY-MM-%"` — verify this works with SQLite's TEXT comparison

---

## Task 3: CLI add, list, categories Commands

**Phase:** 2
**Can run parallel with:** Task 4 (export command)
**Depends on:** Task 1, Task 2

**Objective:**
Implement the `add`, `list`, and `categories` CLI commands using `click`.

**Scope:**
Included:
- `cli.py` entry point with `click` group
- `expense add <amount> <category> [note]`
- `expense list [--month YYYY-MM] [--category <cat>]`
- `expense categories`

Not included:
- `expense export` command (Task 4)

**Input (what must exist before starting):**
- Tasks 1 and 2 complete: `ExpenseDB` with all query methods

**Output (what this task produces):**
- `cli.py` with working `add`, `list`, `categories` commands

**Implementation Method:**
- Chosen approach: Single `@click.group()` in `cli.py`. Each command is a `@cli.command()` decorated function. `ExpenseDB` instantiated once per command invocation using `get_config()`.
- Key library: `click` — use `@click.argument()` for positional args, `@click.option()` for flags.

**Interface Contract:**
```
expense add <amount: float> <category: str> [note: str]
expense list [--month YYYY-MM] [--category <str>]
expense categories
```

**Constraints:**
- `category` in `add` must be validated against the categories list from config — raise `click.BadParameter` if invalid
- `list` output must be a formatted table (use Python's `tabulate` or manual column formatting — no new dependencies unless `tabulate` is already in the project)
- Amount must be a positive float — validate in the command, not in `db.py`

**Context (for subagent):**
Config shape: `{ "categories": ["food", "transport", ...], "db_path": str }`
ExpenseDB interface: see Task 2 interface contract.

**Risks / Unknowns:**
- `tabulate` is not in the stdlib — if a clean table is needed, either use manual f-string formatting or confirm `tabulate` can be added as a dependency

---

## Task 4: export Command and CSV Module

**Phase:** 3
**Can run parallel with:** none (depends on Task 3's CLI structure)
**Depends on:** Task 2, Task 3

**Objective:**
Implement `export.py` and wire the `expense export` command into the CLI.

**Scope:**
Included:
- `export.py` with `export_to_csv(rows, output_path)` function
- `expense export [--month YYYY-MM] [--out <file>]` command in `cli.py`

Not included:
- Any DB query changes
- Other export formats

**Input (what must exist before starting):**
- Task 2: `get_expenses()` returns `list[dict]`
- Task 3: `cli.py` with `click` group already established

**Output (what this task produces):**
- `export.py`
- Updated `cli.py` with `export` command added

**Implementation Method:**
- Chosen approach: `csv.DictWriter` with fieldnames `["id", "amount", "category", "note", "created_at"]`. Output path defaults to `./expenses-{YYYY-MM}.csv` if `--out` not specified.
- Key library: `csv` stdlib — no new dependency

**Interface Contract:**
```python
# export.py
def export_to_csv(rows: list[dict], output_path: str) -> None: ...
```

**Constraints:**
- Must not overwrite an existing file without warning — check `os.path.exists()` first and prompt the user or add `--force` flag
- CSV must include a header row

**Context (for subagent):**
`get_expenses()` returns: `[{ "id": int, "amount": float, "category": str, "note": str|None, "created_at": str }]`

**Risks / Unknowns:**
- File overwrite behavior is a deferred decision — implementer should choose: prompt, --force flag, or error. Document the choice.

---

---

## Example 2: Larger System — AI-Powered Slack Standup Summarizer

### Input Spec (from idea-refiner output)

> **System Components:**
> - `scheduler.js`: Cron jobs for morning prompt and digest time
> - `slackBot.js`: Slack Bolt app, handles DM events and posting
> - `replyStore.js`: In-memory store of today's replies
> - `summarizer.js`: Calls Claude API, produces structured summary with blocker flags
> - `config.js`: Team member list, channel ID, prompt/digest times
>
> **Tech Stack:** Node.js, `@slack/bolt`, `node-cron`, `@anthropic-ai/sdk`
>
> **Key Data Flow:**
> 1. Cron fires at 09:00 → DM each team member with standup prompt
> 2. Team member replies → stored in replyStore
> 3. Cron fires at 10:00 → collect replies → call Claude → post digest → clear store

---

## Implementation Plan

### Architecture Summary
A Slack bot that sends morning standup prompts to a configured team list, collects replies via DM, generates an AI summary with blocker detection using Claude, and posts the digest to a Slack channel at a fixed time. Runs as a single always-on Node.js service with no external database (in-memory store for v1).

### Stack Confirmation
- Runtime: Node.js 18+ (LTS)
- Slack SDK: `@slack/bolt` v3
- Scheduler: `node-cron`
- AI: `@anthropic-ai/sdk` with claude-sonnet-4-6
- Storage: In-memory Map (v1) — no external DB
- Config: Environment variables + `config.js` constants
- Deploy: Any environment that can run a persistent Node.js process

### Component Analysis

**config.js**
- Chosen approach: Export a frozen config object built from env vars + hardcoded constants. Validates required env vars at startup (fail fast if missing).
- Why: Mixing env vars (secrets) with hardcoded constants (team list, times) is cleaner than putting everything in env. Validates at startup so errors surface immediately, not mid-operation.
- Key risks: Team list hardcoded in config.js means code changes needed to add/remove members — acceptable for v1 but flag as a known limitation.

**replyStore.js**
- Chosen approach: Module-level `Map<string, { text: string, receivedAt: Date }>` with simple get/set/clear methods. No persistence layer.
- Why: Simplest correct solution for v1. Replies only need to live for a few hours per day. A database would add complexity with zero v1 benefit.
- Key risks: Server restart between 09:00 and 10:00 loses all collected replies. Document this clearly in the README.

**slackBot.js**
- Chosen approach: `@slack/bolt` App instance. Handles `message` events in app DMs to capture replies. Exposes `sendDM(userId, text)` and `postToChannel(channelId, text)` functions.
- Why: Bolt abstracts Slack's event API, OAuth, and signature verification. Raw HTTP would require reimplementing all of that. Bolt is the official, maintained approach.
- Key risks: Slack bot permissions setup (scopes) must be correct — `chat:write`, `im:write`, `im:history`, `users:read`. Missing scopes cause cryptic 403 errors at runtime, not at startup.

**scheduler.js**
- Chosen approach: Two `node-cron` jobs — one at 09:00 (send prompts), one at 10:00 (collect + summarize + post). Cron times read from config.
- Why: `node-cron` is lightweight and well-understood. The two-cron pattern maps cleanly to the two operations.
- Key risks: Timezone handling — `node-cron` uses the server's system timezone by default. If the server timezone differs from the team's timezone, digests fire at wrong times. Must set timezone explicitly.

**summarizer.js**
- Chosen approach: Single `summarizeReplies(replies)` function using `@anthropic-ai/sdk`. Uses claude-sonnet-4-6. Structured prompt with explicit blocker detection instructions.
- Why: Claude is the specified AI. Sonnet is the right balance of quality and cost for this task. A single function keeps the concern isolated.
- Key risks: Prompt quality for blocker detection needs tuning — first version may over- or under-flag blockers. Plan for prompt iteration. Also: Claude API latency (~2–5s) is acceptable here since the digest is posted at a scheduled time, not in response to a user action.

**index.js (entry point)**
- Chosen approach: Single entry point that initializes config validation, starts Bolt app, and registers cron jobs. Minimal logic — just wiring.
- Why: Separating the entry point from the bot and scheduler lets each module be tested independently.
- Key risks: Startup order matters — Bolt must be initialized before cron jobs fire.

### Dependency Map
```
config.js       → independent
replyStore.js   → independent
slackBot.js     → needs → config.js (for bot token, signing secret)
summarizer.js   → needs → config.js (for Claude API key)
scheduler.js    → needs → slackBot.js, replyStore.js, summarizer.js, config.js
index.js        → needs → all of the above (wiring only)
```

### Build Phases
- **Phase 1 (Foundation):** config.js, replyStore.js
- **Phase 2 (Core):** slackBot.js, summarizer.js
- **Phase 3 (Integration):** scheduler.js, index.js (full wiring + end-to-end test)
- **Phase 4 (Secondary):** Error handling hardening, logging, Slack message formatting polish

### Parallelization Opportunities
- Task 3 (slackBot.js) and Task 4 (summarizer.js) can run in parallel — both depend only on Phase 1 outputs and have no shared interface until Phase 3.

---

### Task Packages

---

## Task 1: Config Module with Validation

**Phase:** 1
**Can run parallel with:** Task 2 (replyStore.js)
**Depends on:** none

**Objective:**
Create `config.js` that loads all required configuration from environment variables, validates at startup, and exports a single frozen config object used by all other modules.

**Scope:**
Included:
- `config.js` with all config values
- Startup validation that throws on missing required env vars
- `.env.example` with all required variable names documented

Not included:
- Any Slack or Claude initialization (those modules own their own clients)
- Dynamic config reloading

**Input (what must exist before starting):**
- Node.js 18+ project initialized with `package.json`
- `dotenv` package available

**Output (what this task produces):**
- `config.js`
- `.env.example`

**Implementation Method:**
- Chosen approach: Load env vars via `dotenv` at the top of `config.js`. Validate with a simple check: throw a descriptive error for each missing required var. Export `Object.freeze({...})`.
- Why: `Object.freeze` prevents accidental mutation of config across modules.

**Interface Contract:**
```javascript
// config.js exports:
module.exports = {
  slack: {
    botToken: string,       // SLACK_BOT_TOKEN
    signingSecret: string,  // SLACK_SIGNING_SECRET
    digestChannelId: string // SLACK_DIGEST_CHANNEL_ID
  },
  claude: {
    apiKey: string          // ANTHROPIC_API_KEY
  },
  schedule: {
    promptTime: string,     // STANDUP_PROMPT_TIME — cron format e.g. "0 9 * * 1-5"
    digestTime: string,     // STANDUP_DIGEST_TIME — cron format e.g. "0 10 * * 1-5"
    timezone: string        // SCHEDULE_TIMEZONE e.g. "Asia/Taipei"
  },
  team: {
    members: { userId: string, name: string }[]  // TEAM_MEMBERS — JSON array in env var
  }
}
```

**Constraints:**
- Must validate ALL required env vars at startup before exporting — do not let missing vars cause runtime errors deep in the call stack
- `TEAM_MEMBERS` is a JSON string in the env var — parse it and validate it's an array

**Context (for subagent):**
Required env vars:
- `SLACK_BOT_TOKEN` — Slack bot OAuth token (starts with `xoxb-`)
- `SLACK_SIGNING_SECRET` — from Slack app settings
- `SLACK_DIGEST_CHANNEL_ID` — channel where daily digest is posted
- `ANTHROPIC_API_KEY` — Claude API key
- `STANDUP_PROMPT_TIME` — cron expression for when prompts are sent
- `STANDUP_DIGEST_TIME` — cron expression for when digest is posted
- `SCHEDULE_TIMEZONE` — IANA timezone name
- `TEAM_MEMBERS` — JSON string: `[{"userId":"U123","name":"Alice"},...]`

**Risks / Unknowns:**
- `TEAM_MEMBERS` as a JSON env var is awkward to set in some deployment environments — flag this as a v1 limitation worth revisiting

---

## Task 2: Reply Store Module

**Phase:** 1
**Can run parallel with:** Task 1 (config.js)
**Depends on:** none

**Objective:**
Create `replyStore.js`, an in-memory store that holds standup replies for the current day.

**Scope:**
Included:
- `replyStore.js` with store, get, set, clear, and getAll methods

Not included:
- Persistence to disk or database
- Any Slack integration

**Input (what must exist before starting):**
- Node.js 18+ project initialized

**Output (what this task produces):**
- `replyStore.js`

**Implementation Method:**
- Chosen approach: Module-level `Map<userId: string, { text: string, receivedAt: Date }>`. No class needed — a module with exported functions is simpler.
- Why: A Map gives O(1) lookup by userId. Module-level state is fine for a single-process Node app.

**Interface Contract:**
```javascript
// replyStore.js exports:
function storeReply(userId, text)           // void
function getReply(userId)                   // { text: string, receivedAt: Date } | undefined
function getAllReplies()                     // { userId: string, text: string, receivedAt: Date }[]
function clearReplies()                     // void — call after digest is posted
function hasReply(userId)                   // boolean
```

**Constraints:**
- Must be stateless across module reloads — no global singleton pattern that breaks in tests
- `clearReplies()` must reset completely — no stale entries carry over to the next day

**Context (for subagent):**
This module is used by `scheduler.js` (to read all replies at digest time) and `slackBot.js` (to store incoming replies). It has no dependencies on other modules.

**Risks / Unknowns:**
- In-memory state is lost on server restart — this is an accepted v1 limitation. Document it.
- If a team member submits multiple replies (edits, follow-ups), the latest reply overwrites the earlier one. This is acceptable behavior for v1.

---

## Task 3: Slack Bot Module

**Phase:** 2
**Can run parallel with:** Task 4 (summarizer.js)
**Depends on:** Task 1 (config.js)

**Objective:**
Create `slackBot.js` that initializes the Slack Bolt app, captures incoming DM replies, and exposes functions for sending DMs and posting to a channel.

**Scope:**
Included:
- Bolt app initialization with bot token and signing secret from config
- `message` event handler for DMs (captures standup replies)
- `sendPrompt(userId, text)` — sends a DM to a team member
- `postDigest(channelId, text)` — posts to a channel
- `startBot()` — starts the Bolt receiver

Not included:
- Cron scheduling (Task 5)
- Reply storage logic (calls into replyStore.js — that module is separate)
- Summarization logic

**Input (what must exist before starting):**
- Task 1 complete: `config.js` exports `slack.botToken`, `slack.signingSecret`
- Task 2 complete: `replyStore.js` with `storeReply()` function
- `@slack/bolt` installed

**Output (what this task produces):**
- `slackBot.js` with initialized Bolt app and all exported functions

**Implementation Method:**
- Chosen approach: `new App({ token, signingSecret })` from `@slack/bolt`. Register `app.message()` for DM events. Filter to only capture direct messages using the event's `channel_type === 'im'` check.
- Key library: `@slack/bolt` v3

**Interface Contract:**
```javascript
// slackBot.js exports:
async function startBot()                        // Starts Bolt HTTP receiver on configured port
async function sendPrompt(userId, promptText)    // DMs a user; returns void
async function postDigest(channelId, text)       // Posts formatted message to channel; returns void
// Internal: registers app.message() handler that calls replyStore.storeReply()
```

**Constraints:**
- DM handler must only capture messages from team members listed in config — ignore DMs from other users
- Must not acknowledge Slack events with an error — always acknowledge even if processing fails (Slack will retry if not acknowledged within 3s)
- Port for Bolt receiver should come from `PORT` env var with fallback to 3000

**Context (for subagent):**
Config available: `config.slack.botToken`, `config.slack.signingSecret`
replyStore interface: `storeReply(userId: string, text: string): void`
Team members: `config.team.members` — array of `{ userId, name }`

Required Slack bot scopes (must be configured in Slack app settings):
- `chat:write` — send messages
- `im:write` — open DM channels
- `im:history` — read DM history (needed for message events)

**Risks / Unknowns:**
- Slack `im:history` scope may need explicit user grant depending on workspace settings — test with a real workspace before assuming it works
- Bolt v3 Socket Mode vs HTTP mode: default is HTTP. For local dev without a public URL, Socket Mode is easier. Consider flagging this as a dev-environment decision.

---

## Task 4: Summarizer Module

**Phase:** 2
**Can run parallel with:** Task 3 (slackBot.js)
**Depends on:** Task 1 (config.js)

**Objective:**
Create `summarizer.js` that takes collected standup replies and returns a structured summary with blockers flagged, using the Claude API.

**Scope:**
Included:
- `summarizeReplies(replies)` function
- Prompt construction
- Claude API call using `@anthropic-ai/sdk`
- Parsing Claude's response into structured output

Not included:
- Slack posting (that's slackBot.js)
- Any scheduling

**Input (what must exist before starting):**
- Task 1 complete: `config.js` exports `claude.apiKey`
- `@anthropic-ai/sdk` installed

**Output (what this task produces):**
- `summarizer.js`

**Implementation Method:**
- Chosen approach: Use `@anthropic-ai/sdk` `messages.create()` with claude-sonnet-4-6. Construct a prompt that lists all replies and explicitly asks Claude to: (1) write a brief team summary, (2) list any blockers separately. Parse the response as plain text — no JSON structured output needed for v1.
- Key library: `@anthropic-ai/sdk`

**Interface Contract:**
```javascript
// summarizer.js exports:
async function summarizeReplies(replies)
// replies: [{ userId: string, name: string, text: string }]
// returns: { summary: string, blockers: string[] }
// blockers is [] if none detected
```

**Constraints:**
- Must handle Claude API errors gracefully — if the API call fails, return `{ summary: "Summary unavailable — API error.", blockers: [] }` rather than crashing
- Model must be `claude-sonnet-4-6` (not hardcoded as a string — use a constant)
- Use prompt caching headers if input replies are long (implement via `@anthropic-ai/sdk` cache_control)

**Context (for subagent):**
Suggested prompt structure:
```
You are summarizing daily standup updates for a software team.

Here are today's updates:
${replies.map(r => `${r.name}: ${r.text}`).join('\n')}

Please:
1. Write a brief team summary (2-4 sentences)
2. List any blockers mentioned (if none, say "No blockers")

Format:
Summary: [your summary]
Blockers:
- [blocker 1]
- [blocker 2]
(or "None" if no blockers)
```

**Risks / Unknowns:**
- Claude's blocker detection quality depends on how explicitly team members mention blockers — the prompt may need tuning after real usage
- Response parsing assumes a consistent format — add a fallback if Claude returns something unexpected

---

## Task 5: Scheduler and Entry Point Wiring

**Phase:** 3
**Can run parallel with:** none
**Depends on:** Tasks 1, 2, 3, 4

**Objective:**
Create `scheduler.js` with the two cron jobs (prompt send, digest collect+post) and `index.js` that wires all modules together and starts the service.

**Scope:**
Included:
- `scheduler.js` with morning prompt cron and digest cron
- `index.js` entry point that initializes config, starts bot, registers cron jobs
- End-to-end wiring of all modules

Not included:
- Any new module logic — this task only wires existing modules
- Message formatting polish (Task 6)

**Input (what must exist before starting):**
- All of Tasks 1–4 complete
- `node-cron` installed

**Output (what this task produces):**
- `scheduler.js`
- `index.js`
- A running service that executes the full standup flow

**Implementation Method:**
- Chosen approach: Two `cron.schedule()` calls in `scheduler.js`. First fires `sendPrompts()` (loops over team members, calls `slackBot.sendPrompt()`). Second fires `sendDigest()` (reads replyStore, calls summarizer, calls slackBot.postDigest(), clears store). `index.js` calls `config` validation, `slackBot.startBot()`, then `scheduler.registerJobs()`.
- Key library: `node-cron` — pass timezone from config to `cron.schedule()`

**Interface Contract:**
```javascript
// scheduler.js exports:
function registerJobs()  // Registers both cron jobs; returns void

// index.js — no exports, entry point only
// Startup order: validate config → start bot → register cron jobs → log ready
```

**Constraints:**
- Cron timezone must be explicitly set from `config.schedule.timezone` — do not rely on server system timezone
- Startup must log the next scheduled fire time for both cron jobs so operators can verify configuration
- `sendDigest()` must call `replyStore.clearReplies()` AFTER the digest is posted, not before

**Context (for subagent):**
Module interfaces needed:
- `slackBot.startBot()` — async, starts Bolt HTTP receiver
- `slackBot.sendPrompt(userId, text)` — async
- `slackBot.postDigest(channelId, text)` — async
- `replyStore.getAllReplies()` — returns `[{ userId, name, text }]`
- `replyStore.clearReplies()` — void
- `summarizer.summarizeReplies(replies)` — async, returns `{ summary, blockers }`
- Config: `config.team.members`, `config.schedule.promptTime`, `config.schedule.digestTime`, `config.schedule.timezone`, `config.slack.digestChannelId`

**Risks / Unknowns:**
- If `slackBot.startBot()` fails (port conflict, bad token), the cron jobs should not register — startup must be sequential with error handling
- `getAllReplies()` enriching userId with name requires cross-referencing `config.team.members` — implement this lookup in scheduler.js before passing to summarizer

---

## Task 6: Message Formatting and Error Hardening

**Phase:** 4
**Can run parallel with:** none
**Depends on:** Task 5

**Objective:**
Polish the Slack digest message formatting and add error handling for the most likely failure modes.

**Scope:**
Included:
- Formatted Slack Block Kit digest message (header, team summary, blockers section)
- Graceful handling of: zero replies received, Claude API timeout, Slack post failure
- Logging for key events (prompt sent, reply received, digest posted, errors)

Not included:
- Web dashboard, admin UI, or analytics
- Timezone per-user configuration
- Retry logic for failed DMs

**Input (what must exist before starting):**
- Task 5 complete: full working pipeline

**Output (what this task produces):**
- Updated `slackBot.js` with Block Kit formatted `postDigest()`
- Updated `scheduler.js` with zero-reply handling
- Console logging throughout the flow

**Implementation Method:**
- Chosen approach: Slack Block Kit for the digest message — header block + section block for summary + section block for blockers. Zero-reply case: post a simple "No standup updates received today." message instead of calling Claude.
- Why: Block Kit messages are visually structured and look professional. Zero-reply short-circuit avoids a meaningless Claude API call.

**Interface Contract:**
No interface changes — this task modifies internals only.

**Constraints:**
- Must not change any existing function signatures (this is polish, not redesign)
- Log format: `[TIMESTAMP] [LEVEL] [MODULE] message` — use `console.log` with ISO timestamps for v1

**Context (for subagent):**
Block Kit structure for the digest:
```json
{
  "blocks": [
    { "type": "header", "text": { "type": "plain_text", "text": "Daily Standup — YYYY-MM-DD" } },
    { "type": "section", "text": { "type": "mrkdwn", "text": "*Summary*\n{summary}" } },
    { "type": "section", "text": { "type": "mrkdwn", "text": "*Blockers*\n{blockers or 'None'}" } }
  ]
}
```

**Risks / Unknowns:**
- Block Kit rendering varies slightly between Slack clients — test on desktop and mobile if possible

---

### Integration Checklist
- `replyStore.js` ↔ `slackBot.js`: DM message handler must call `storeReply()` — verify userId from Slack event matches userId format in `config.team.members`
- `replyStore.js` ↔ `scheduler.js`: `getAllReplies()` returns userIds only — scheduler must enrich with names from config before passing to summarizer
- `summarizer.js` ↔ `slackBot.js`: summarizer returns `{ summary, blockers[] }` — scheduler must format this into the Block Kit message before calling `postDigest()`
- `replyStore.clearReplies()` timing: must happen AFTER `postDigest()` resolves, not before — verify the await chain in scheduler
- Bolt startup timing: cron jobs must not be registered until `startBot()` resolves — the event handler won't work if the bot isn't connected yet

### Pre-Build Decisions

**Decision 1: Slack Bot Connection Mode**
- What needs to be decided: HTTP mode (requires public URL) vs Socket Mode (works without one)
- Why it matters: Affects `slackBot.js` initialization and local development workflow. HTTP needs a tunnel (ngrok) for local dev; Socket Mode works out of the box locally but requires an additional Slack app configuration.
- Options: (A) HTTP mode — production-ready, requires public URL; (B) Socket Mode — easier local dev, slightly different Bolt initialization

**Decision 2: Team Member Management**
- What needs to be decided: Keep team list as a JSON env var (v1) or move it to a JSON file committed to the repo?
- Why it matters: Affects Task 1 (config.js). JSON env vars are awkward to set in some deployment environments.
- Options: (A) JSON env var — no new files, works in all deployment platforms; (B) `team.json` file — easier to edit, visible in version control

**Decision 3: What to do if digest posting fails**
- What needs to be decided: Should `replyStore.clearReplies()` still be called if `postDigest()` throws?
- Why it matters: If clear happens before post and post fails, replies are lost. If clear doesn't happen after a failed post, the next day's digest includes yesterday's replies.
- Options: (A) Clear only on success — safest, but stale replies may accumulate; (B) Always clear — simpler, accepts the data loss risk on rare API failures
