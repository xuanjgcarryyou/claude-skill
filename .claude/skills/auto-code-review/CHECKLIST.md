# Auto Code Review — Extended Checklist

Use this for deep review passes when the standard 7-axis review in SKILL.md needs more detail.
Organized by axis, with per-language callouts where behavior differs.

---

## Axis 1: Dead Code / Unused Items

### All Languages
- [ ] Every import/require/use statement is referenced at least once in the generated code
- [ ] Every declared variable is read at least once
- [ ] Every declared function/method is called or exported
- [ ] Every declared constant is used
- [ ] No `console.log`, `print()`, `fmt.Println`, `debugger`, `pp`, `dump()`, `var_dump()` left in non-debug code
- [ ] No commented-out code blocks (not counting intentionally disabled feature flags)
- [ ] No `// TODO`, `# TODO`, `/* FIXME */` left in production-path code unless it was pre-existing
- [ ] No placeholder return values (`return null`, `return {}`, `pass`, `todo!()`) left in implemented functions
- [ ] No scaffold artifacts (`MyServiceImpl`, `ExampleHandler`, generic test data with `"foo"`/`"bar"`)

### TypeScript / JavaScript
- [ ] No unused type aliases or interfaces
- [ ] No unused enum variants
- [ ] `@ts-ignore` and `@ts-expect-error` removed unless intentional and commented
- [ ] No `any` introduced where a proper type could be used

### Python
- [ ] No unused `import` or `from X import Y` lines
- [ ] No unused `*args` / `**kwargs` in non-interface implementations
- [ ] No bare `except:` clauses left from debugging
- [ ] No `breakpoint()` or `pdb.set_trace()` calls

### Go
- [ ] No unused imports (Go compiler enforces this, but check for `_` blank imports added just to silence errors)
- [ ] No `log.Println` / `fmt.Printf` debug statements in handler paths
- [ ] No unhandled `err` assigned to `_` where it should be handled

### Java / Kotlin
- [ ] No `System.out.println` / `println()` in service code
- [ ] No unused `@Autowired` fields
- [ ] No unused private methods

---

## Axis 2: Redundancy / Duplication

### All Languages
- [ ] Check if the standard library already provides the implemented utility (string manipulation, date formatting, URL parsing, UUID generation, base64 encoding, JSON serialization)
- [ ] Check if the project already has a utility for this — look for `utils/`, `helpers/`, `lib/`, `shared/` directories
- [ ] Check if any logic block appears more than once in the generated code and could be a shared function
- [ ] Check if error handling boilerplate is repeated where a wrapper/middleware would be cleaner

### TypeScript / JavaScript
- [ ] Not reimplementing `Array.prototype` methods (`map`, `filter`, `reduce`, `find`, `some`, `every`)
- [ ] Not reimplementing `Object.entries`, `Object.keys`, `Object.fromEntries`
- [ ] Not reimplementing `Promise.all` / `Promise.allSettled` patterns
- [ ] Not re-implementing lodash/underscore utilities if lodash is already in the project

### Python
- [ ] Not reimplementing `itertools`, `functools`, `collections`, `pathlib` functionality
- [ ] Not reimplementing list comprehensions with explicit loops when comprehension is cleaner
- [ ] Not reimplementing `dataclasses` or `NamedTuple` with manual `__init__`

### Go
- [ ] Not reimplementing `strings`, `strconv`, `bytes`, `io`, `os` standard library functions
- [ ] Not reimplementing `sync.Once` or `sync.Map` patterns manually

---

## Axis 3: Technology Consistency

### All Languages
- [ ] HTTP client matches what the project uses (axios vs fetch vs got vs superagent; requests vs httpx vs urllib3)
- [ ] Logger matches what the project uses (winston vs pino vs console; structlog vs logging; zap vs logrus vs slog)
- [ ] Test assertion style matches (jest vs vitest vs chai; pytest vs unittest; testify vs std testing)
- [ ] Date/time library matches (dayjs vs date-fns vs luxon; arrow vs pendulum vs datetime)
- [ ] Validation library matches (zod vs yup vs joi; pydantic vs marshmallow; validator vs ozzo)

### TypeScript / JavaScript
- [ ] Not mixing CommonJS `require()` with ES Module `import` in the same file
- [ ] Not mixing `.then()` chains with `async/await` in the same function
- [ ] Not mixing named exports and default exports inconsistently with the project pattern
- [ ] Not using `var` in a codebase that uses `const`/`let`
- [ ] Not using `function` declarations where the project uses arrow functions (or vice versa)
- [ ] Not using `interface` vs `type` inconsistently with project convention
- [ ] Not mixing string concatenation with template literals inconsistently

### Python
- [ ] Not mixing `f-string`, `%` formatting, and `.format()` in the same file
- [ ] Not using `typing` module types (`List`, `Dict`, `Optional`) in projects targeting Python 3.10+ (prefer `list`, `dict`, `X | None`)
- [ ] Not using synchronous I/O in an `async` context
- [ ] Not mixing `async def` and `def` in the same class when the project is consistently async

### Go
- [ ] Not mixing custom error types with `fmt.Errorf` wrapping inconsistently
- [ ] Not using `goroutine` + `channel` where the project uses `errgroup` or `sync.WaitGroup`
- [ ] Not mixing struct-based configs with environment variable reads at the handler level

---

## Axis 4: Version / Compatibility

### All Languages
- [ ] Check `package.json` / `go.mod` / `pyproject.toml` / `pom.xml` for declared runtime/language version
- [ ] No syntax or API from a newer version than what is declared

### TypeScript / JavaScript
- [ ] Not using `Array.at()` (Node 16.6+), `structuredClone()` (Node 17+), `Object.hasOwn()` (Node 16.9+) without version confirmation
- [ ] Not using optional chaining `?.` / nullish coalescing `??` without confirming tsconfig target
- [ ] Not using top-level `await` without confirming module type is `"module"` or `"esnext"`
- [ ] Not using `Promise.any()` (ES2021) in projects targeting ES2019 or lower

### Python
- [ ] Not using `match` statement (3.10+) without version confirmation
- [ ] Not using `X | Y` union types (3.10+) without version confirmation
- [ ] Not using `asyncio.TaskGroup` (3.11+) without version confirmation
- [ ] Not using `tomllib` (3.11+) without version confirmation

### Go
- [ ] Not using `slices` or `maps` packages (1.21+) without version confirmation
- [ ] Not using `log/slog` (1.21+) without version confirmation
- [ ] Not using generics (1.18+) without version confirmation

---

## Axis 5: Resource / Performance

### All Languages
- [ ] No DB query inside a loop — should batch or use `WHERE IN`
- [ ] No external HTTP call inside a loop — should batch or parallelize
- [ ] No file open without close / no resource allocation without cleanup path
- [ ] No expensive recomputation in a hot path that could be hoisted or memoized
- [ ] No synchronous blocking call inside an async/event-loop context

### TypeScript / JavaScript
- [ ] No `await` inside `forEach` — use `Promise.all` + `map` instead
- [ ] No `JSON.parse(JSON.stringify(x))` deep clone in a performance-sensitive path
- [ ] No `document.querySelector` in a render loop
- [ ] `setInterval` / `setTimeout` has a corresponding cleanup (in React: `useEffect` return, in Node: `clearInterval`)
- [ ] No large library imported for a single utility (e.g., importing all of `lodash` for `_.get`)

### Python
- [ ] No synchronous `requests` call inside `async def` — use `httpx` or `aiohttp`
- [ ] No `open(file)` without `with` statement
- [ ] No `time.sleep()` inside an async context — use `asyncio.sleep()`
- [ ] Generator expressions used for large data sequences where possible

### Go
- [ ] `defer` used for resource cleanup (file close, DB connection release, mutex unlock)
- [ ] No goroutine leak — every spawned goroutine has a clear exit condition
- [ ] No unbuffered channel used where blocking is unintentional

---

## Axis 6: Config / Environment Hygiene

### All Languages
- [ ] No hardcoded IP addresses, hostnames, or port numbers
- [ ] No hardcoded credentials, API keys, tokens, or secrets
- [ ] No hardcoded environment names (`"production"`, `"staging"`, `"dev"`) in business logic
- [ ] No hardcoded file paths that are OS-specific or user-specific
- [ ] Magic numbers (timeouts, limits, retry counts, thresholds) extracted to named constants
- [ ] Magic strings (status codes, event names, queue names) extracted to named constants or enums
- [ ] Environment-specific behavior reads from config/env, not from inline conditionals

### TypeScript / JavaScript
- [ ] No `process.env.X` read inline without a config module or validation layer
- [ ] No hardcoded `localhost:3000` or similar URLs
- [ ] Constants file or enum used for repeated string literals

### Python
- [ ] No `os.environ["X"]` inline without a settings/config module
- [ ] No hardcoded database DSNs or connection strings
- [ ] `pydantic-settings` / `dynaconf` / `python-decouple` pattern used if project already has one

### Go
- [ ] No hardcoded config in `main.go` — use struct-based config loaded from env/file
- [ ] No `os.Getenv("X")` scattered across handlers — centralize in a config struct

---

## Axis 7: Naming / Interface Consistency

### All Languages
- [ ] Function/method names match the verb style of existing code (get vs fetch vs load vs read)
- [ ] Boolean variable/parameter names use `is`/`has`/`should`/`can` prefix consistently with codebase
- [ ] Error variable names match convention (`err`, `error`, `e`, `ex` — pick what the project uses)
- [ ] Test function/describe block naming matches project convention
- [ ] Return values structured consistently with similar functions (object vs tuple vs multiple returns)
- [ ] Error handling pattern matches project convention (throw vs return error vs Result type vs Either)

### TypeScript / JavaScript
- [ ] File naming matches project convention (`camelCase.ts` vs `kebab-case.ts` vs `PascalCase.ts`)
- [ ] Class naming is PascalCase, consistent with project
- [ ] Interface/type naming does not use `I` prefix unless project convention requires it
- [ ] Callback and event handler naming matches project (`handleX`, `onX`, `_onX`)

### Python
- [ ] `snake_case` used for functions, variables, modules — consistent with PEP 8 and project
- [ ] `PascalCase` used for classes
- [ ] Private methods prefixed with `_` if project convention uses it
- [ ] Async functions named consistently (some projects suffix with `_async` or `Async`)

### Go
- [ ] Exported names are PascalCase, unexported are camelCase
- [ ] Error type names end in `Error` if the project uses that convention
- [ ] Interface names end in `-er` (Reader, Writer, Handler) if project uses that convention
- [ ] Receiver variable names are short (1–2 letters) and consistent with the project
