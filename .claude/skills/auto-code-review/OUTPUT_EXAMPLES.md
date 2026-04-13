# Auto Code Review — Output Examples

Three realistic examples showing Claude reviewing its own generated code.
Each shows the generated code followed by the self-review report appended to the response.

---

## Example 1: TypeScript Service (Email Notification Service)

**User prompt:** "Write an email notification service that sends a welcome email when a new user registers."

**Claude's generated code:**

```typescript
import nodemailer from 'nodemailer';
import axios from 'axios';
import _ from 'lodash';
import { User } from '../models/user';
import { logger } from '../utils/logger';
import { db } from '../db';

const SMTP_HOST = 'smtp.mailgun.org';
const SMTP_PORT = 587;
const FROM_EMAIL = 'noreply@myapp.com';

export async function sendWelcomeEmail(user: User): Promise<void> {
  const transporter = nodemailer.createTransporter({
    host: SMTP_HOST,
    port: SMTP_PORT,
    auth: {
      user: 'api',
      pass: 'key-abc123secrettoken',
    },
  });

  const userName = _.get(user, 'name', 'there');

  const allUsers = await db.query('SELECT * FROM users');
  console.log('Sending welcome email to:', user.email);

  await transporter.sendMail({
    from: FROM_EMAIL,
    to: user.email,
    subject: 'Welcome!',
    html: `<p>Hi ${userName}, welcome to the app!</p>`,
  });

  logger.info('Welcome email sent', { userId: user.id });
}

// TODO: add retry logic
// function retryEmail() {}
```

---

### Self-Review

**Verdict:** `ISSUES FOUND`

| # | Severity | Axis | Finding | Fix |
|---|----------|------|---------|-----|
| 1 | REMOVE | Axis 1: Dead Code | `axios` imported but never used | Remove `import axios from 'axios'` |
| 2 | REMOVE | Axis 1: Dead Code | `console.log` debug statement on line 21 | Remove the `console.log` line |
| 3 | REMOVE | Axis 1: Dead Code | Commented-out `retryEmail` function and `// TODO` left in code | Remove both lines |
| 4 | REMOVE | Axis 6: Config Hygiene | SMTP password `'key-abc123secrettoken'` hardcoded inline | Replace with `process.env.SMTP_PASSWORD` (or project config module) |
| 5 | REFACTOR | Axis 2: Redundancy | `_.get(user, 'name', 'there')` — lodash imported for a single optional access | Replace with `user.name ?? 'there'`; remove lodash import |
| 6 | REFACTOR | Axis 5: Resource/Performance | `nodemailer.createTransporter` called on every invocation | Hoist transporter to module-level singleton |
| 7 | VERIFY | Axis 4: Version/Compatibility | `db.query('SELECT * FROM users')` fetches all users but result is unused | Appears to be leftover debug code — confirm and remove if so |
| 8 | NOTE | Axis 6: Config Hygiene | `SMTP_HOST`, `SMTP_PORT`, `FROM_EMAIL` as file-level constants — acceptable if single-use, but should move to config module if project has one | Move to config if `src/config/` exists in the project |

**Confidence Notes:**
- Could not confirm whether the project already has a transporter/mailer utility — if `src/services/mailer.ts` exists, this function may be duplicating it.
- `db.query` usage pattern unconfirmed — could not verify if project uses query builder or ORM instead.

---

## Example 2: Python Data Pipeline (CSV Ingestion)

**User prompt:** "Write a Python function that reads a CSV of transactions, filters rows where amount > 1000, and saves the result to a new CSV."

**Claude's generated code:**

```python
import csv
import os
import json
import datetime
import pandas as pd
from typing import List, Dict, Any

OUTPUT_DIR = "/tmp/processed"

def process_transactions(input_path: str) -> List[Dict[str, Any]]:
    results = []
    
    with open(input_path, 'r') as f:
        reader = csv.DictReader(f)
        for row in reader:
            amount = float(row['amount'])
            if amount > 1000:
                row['processed_at'] = str(datetime.datetime.now())
                results.append(row)
    
    output_path = OUTPUT_DIR + "/" + "filtered_transactions.csv"
    
    df = pd.DataFrame(results)
    df.to_csv(output_path, index=False)
    
    print(f"Processed {len(results)} rows")
    print(f"Output: {output_path}")
    
    # debug
    # import pdb; pdb.set_trace()
    
    return results


def helper(data):
    return json.dumps(data)
```

---

### Self-Review

**Verdict:** `ISSUES FOUND`

| # | Severity | Axis | Finding | Fix |
|---|----------|------|---------|-----|
| 1 | REMOVE | Axis 1: Dead Code | `os` imported but never used | Remove `import os` |
| 2 | REMOVE | Axis 1: Dead Code | `json` imported but only used in unused `helper()` function | Remove `import json` |
| 3 | REMOVE | Axis 1: Dead Code | `helper(data)` function declared but never called or exported | Remove the function |
| 4 | REMOVE | Axis 1: Dead Code | Commented-out `pdb.set_trace()` debug line | Remove the comment block |
| 5 | REMOVE | Axis 1: Dead Code | `print()` statements in a function that already uses a logging pattern (if project has one) | Replace with `logger.info()` or remove |
| 6 | REFACTOR | Axis 2: Redundancy | `csv.DictReader` used for reading, then `pandas.DataFrame` used for writing — two separate I/O libraries for the same file operation | Use `pd.read_csv` + `df[df['amount'] > 1000].to_csv()` entirely, or use `csv` entirely; mixing both adds unnecessary dependency |
| 7 | REFACTOR | Axis 6: Config Hygiene | `OUTPUT_DIR = "/tmp/processed"` hardcoded absolute path | Accept `output_dir` as a parameter or read from config/env |
| 8 | REFACTOR | Axis 7: Naming Consistency | `helper` is a non-descriptive function name | Rename to `serialize_to_json` or remove if unused |
| 9 | NOTE | Axis 4: Version/Compatibility | `List[Dict[str, Any]]` typing from `typing` module — if project targets Python 3.10+, prefer `list[dict[str, Any]]` | Confirm Python version in `pyproject.toml` or `setup.cfg` |

**Confidence Notes:**
- Could not confirm whether the project has a logging setup — `print()` may be acceptable if this is a standalone script.
- `pandas` may already be a project dependency or may be new — could not verify without seeing `requirements.txt` or `pyproject.toml`.

---

## Example 3: Go HTTP Handler (User Profile Endpoint)

**User prompt:** "Write a Go HTTP handler that fetches a user profile by ID from the database and returns it as JSON."

**Claude's generated code:**

```go
package handlers

import (
	"database/sql"
	"encoding/json"
	"fmt"
	"log"
	"math/rand"
	"net/http"
	"strconv"

	"github.com/myapp/db"
)

type UserProfile struct {
	ID       int    `json:"id"`
	Name     string `json:"name"`
	Email    string `json:"email"`
	Role     string `json:"role"`
}

func GetUserProfileHandler(database *db.DB) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		idStr := r.URL.Query().Get("id")
		id, err := strconv.Atoi(idStr)
		if err != nil {
			http.Error(w, "invalid id", http.StatusBadRequest)
			return
		}

		query := fmt.Sprintf("SELECT id, name, email, role FROM users WHERE id = %d", id)
		row := database.QueryRow(query)

		var profile UserProfile
		err = row.Scan(&profile.ID, &profile.Name, &profile.Email, &profile.Role)
		if err == sql.ErrNoRows {
			http.Error(w, "user not found", http.StatusNotFound)
			return
		}
		if err != nil {
			log.Printf("db error: %v", err)
			http.Error(w, "internal error", http.StatusInternalServerError)
			return
		}

		_ = rand.Int()

		w.Header().Set("Content-Type", "application/json")
		json.NewEncoder(w).Encode(profile)
	}
}
```

---

### Self-Review

**Verdict:** `ISSUES FOUND`

| # | Severity | Axis | Finding | Fix |
|---|----------|------|---------|-----|
| 1 | REMOVE | Axis 1: Dead Code | `math/rand` imported and `rand.Int()` called with result discarded — clearly leftover scaffold | Remove the import and the `_ = rand.Int()` line |
| 2 | REMOVE | Axis 6: Config Hygiene | `fmt.Sprintf("SELECT ... WHERE id = %d", id)` — SQL built via string formatting, not parameterized query | Use `database.QueryRow("SELECT id, name, email, role FROM users WHERE id = ?", id)` (or `$1` for Postgres) — this is a SQL injection risk |
| 3 | REFACTOR | Axis 3: Technology Consistency | `log.Printf` used for error logging — verify this matches the project's logger (zap, logrus, slog) | Replace with project logger if one exists (e.g., `logger.Error("db error", zap.Error(err))`) |
| 4 | VERIFY | Axis 7: Interface Consistency | User ID read from query param `?id=X` — confirm this matches the project's routing convention (may expect path param `/users/:id`) | Check existing route handler patterns in the project |
| 5 | NOTE | Axis 7: Naming Consistency | `GetUserProfileHandler` — verify whether project names handlers with `Handler` suffix or just the action (e.g., `GetUserProfile`, `UserProfileHandler`) | Check other handler files for naming pattern |

**Confidence Notes:**
- SQL placeholder syntax (`?` vs `$1`) depends on the database driver — could not confirm without seeing `go.mod` or existing query patterns.
- Could not verify whether project uses a structured logger — `log.Printf` is standard library and may be intentional in simpler projects.
- `db.DB` type alias assumed — could not confirm the actual type without reading the `db` package.
