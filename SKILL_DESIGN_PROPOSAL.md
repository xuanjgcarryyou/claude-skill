# Vibe Coding Skill 完整技能體系設計提案

> 基於《Vibe Coding Skill 工程規範完全手冊》設計。
> 審閱後確認即可開始實作。

---

## 一、完整技能體系總覽

### 現有技能（已實作，可直接使用）

| 技能 | 對應手冊名稱 | 狀態 |
|------|------------|------|
| `auto-code-review` | reviewing-code | ✅ 現有 |
| `idea-refiner` | refining-ideas | ✅ 現有 |
| `implementation-planner` | planning-implementation | ✅ 現有 |

### 新增技能（本提案設計，待實作）

| # | 技能 | 定位 | 優先級 | 自由度 |
|---|------|------|--------|--------|
| 1 | `setting-up-context` | Session 初始化 | 🔴 最高 | Narrow Bridge |
| 2 | `contracting-api` | API 契約鎖定 | 🟡 高 | Narrow Bridge |
| 3 | `migrating-schema` | DB 遷移安全閘門 | 🔴 最高 | Narrow Bridge |
| 4 | `auditing-dependencies` | 套件健康審查 | 🟡 高 | Narrow Bridge |
| 5 | `reviewing-security` | 深度安全審查 | 🔴 最高 | Narrow Bridge |
| 6 | `writing-tests` | 測試自動生成 | 🟡 高 | Mixed |
| 7 | `refactoring-safely` | 安全重構前置契約 | 🟡 高 | Narrow Bridge |
| 8 | `debugging-systematically` | 系統性根因分析 | 🟡 高 | Narrow Bridge |
| 9 | `summarizing-pr` | PR Description 生成 | 🟢 中 | Narrow Bridge |
| 10 | `running-retrospective` | 會後知識沉澱 | 🟢 中 | Mixed |

### 完整工作流程圖

```
[Session 開始]
    └─ setting-up-context       ← 強制初始化

[規劃階段]
    ├─ idea-refiner              ← 現有
    ├─ implementation-planner    ← 現有
    └─ contracting-api           ← API 契約鎖定

[實作階段]
    ├─ migrating-schema          ← DB 變更安全閘
    └─ auditing-dependencies     ← 套件引入審查

[審查階段]
    ├─ auto-code-review          ← 現有，每次生成後自動
    ├─ reviewing-security        ← 上線前深度安全審查
    └─ writing-tests             ← 補齊測試覆蓋

[維護/Debug 階段]
    ├─ refactoring-safely        ← 重構前置契約
    └─ debugging-systematically  ← 卡住時根因分析

[收尾階段]
    ├─ summarizing-pr            ← 生成 PR Description
    └─ running-retrospective     ← 更新 CLAUDE.md 規則
```

---

## 二、新技能 SKILL.md 設計內容

---

### 1. setting-up-context

```yaml
---
name: setting-up-context
description: >
  Session initializer that reads project configuration files and outputs a confirmed
  Session Brief before any work begins. Prevents context contamination between features
  by grounding every session in the current project state. TRIGGER at session start,
  after /clear, before switching features, or when the user says "let's start a new
  session", "initialize context", "set up context", "read the project files", "what
  are we working on", "what's our stack", "remind yourself of the project", "before
  we start", "new task context", or "fresh start". ALSO trigger automatically when
  Claude detects it is beginning work in a new conversation without having confirmed
  project context. Do NOT trigger during ongoing work within a session that already
  has a confirmed Brief.
---
```

**核心行為（Low Freedom）：**

1. 讀 CLAUDE.md → 若不存在，硬停止並要求補充，不猜測 Stack
2. 讀 PRD.md → 若不存在，在 Session Brief 標記 UNCONFIRMED
3. 讀 TechDesign.md（選擇性）
4. 輸出 Session Brief，**等待確認後才進行任何工作**

**Session Brief 格式（固定六欄）：**
```
## Session Brief
**Stack:** [from CLAUDE.md]
**Build/Test Commands:** [from CLAUDE.md]
**Current Task:** [from PRD.md, or UNCONFIRMED]
**Feature Scope:** [in/out of scope]
**Off-limits Zones:** [from CLAUDE.md]
**Active Constraints:** [naming rules, behavioral constraints]

Ready to proceed. Please confirm or correct any of the above.
```

**硬規則：**
- CLAUDE.md 缺失 → 輸出 CONTEXT SETUP BLOCKED，不繼續
- Brief 確認前禁止提案、設計、寫碼

**驗證摘要：**
- Trigger 衝突風險：低（「初始化」語義與其他 skill 不重疊）
- 自由度：Narrow Bridge，固定六欄格式
- Failure Mode：CLAUDE.md 缺失=硬停止；PRD.md 缺失=繼續但標 UNCONFIRMED

**子文件：** `SESSION_TEMPLATE.md`（手動填寫用模板）、`README.md`

---

### 2. contracting-api

```yaml
---
name: contracting-api
description: >
  Locks API interface contracts before parallel development begins, serving as
  the formal bridge between frontend and backend teams. Invoke this skill when
  a user says "define API", "lock the interface", "API contract",
  "before we implement", or "frontend-backend boundary". It outputs an
  OpenAPI-compatible YAML contract marked LOCKED with a semantic version,
  without implementing any endpoints. Any contract modification requires a
  version bump and explicit notification to all dependent parties.
---
```

**核心行為（Narrow Bridge）：**

1. 收集需求（HTTP method、path、auth、pagination）
2. 輸出 LOCKED YAML contract（v1.0.0）
3. 輸出 Post-Contract Checklist

**Contract 輸出格式：**
```yaml
contract:
  version: v1.0.0
  status: LOCKED
  endpoint:
    method: POST
    path: /api/resource
    auth: Bearer JWT
  request_body:
    required: [...]
    optional: [...]
  responses:
    "201": { body: [...] }
  errors:
    "400": { code: VALIDATION_ERROR }
  changelog:
    - version: v1.0.0
      note: "Initial contract lock"
```

**Version Bump 規則：**
- 新增選填欄 → Minor (v1.1.0)
- 新增必填欄/刪除欄/改類型 → Major (v2.0.0)
- 僅說明修改 → Patch (v1.0.1)

**硬規則：**
- 不實作 endpoints
- 輸入不完整時輸出 `status: DRAFT`，不鎖定
- 每個 endpoint 獨立一個 contract block

**驗證摘要：**
- Trigger 衝突風險：低（"API contract" 語義明確）
- 自由度：Narrow Bridge，格式固定
- Failure Mode：資訊不足 → 輸出 DRAFT，placeholder 標 `<REQUIRED: description>`

---

### 3. migrating-schema

```yaml
---
name: migrating-schema
description: >
  Acts as the mandatory safety gate for all database schema changes, requiring
  explicit approval before any migration file is generated. Invoke this skill
  when a user says "migration", "alter table", "schema change", "add column",
  "drop column", or mentions "schema.prisma". It produces an impact analysis
  with risk level, forward SQL (up migration), rollback SQL (down migration),
  and production recovery steps. HIGH RISK operations — including DROP COLUMN,
  DROP TABLE, and adding NOT NULL constraints — are blocked until a data
  migration strategy is confirmed and the user explicitly approves.
---
```

**核心行為（嚴格 Narrow Bridge）：**

強制七段輸出順序：
1. Impact Analysis（受影響的 table、code、預估停機）
2. Risk Explanation（plain English 說明風險）
3. Data Migration Strategy（**HIGH RISK 才有，必填**）
4. Forward SQL（up migration，含 transaction）
5. Rollback SQL（down migration，不可省略）
6. Production Recovery Steps（編號 checklist）
7. ⚠️ Awaiting Approval Gate → **停止，等待確認**

**Risk 分類（自動）：**

| 操作 | 預設風險 |
|------|---------|
| ADD COLUMN（nullable） | LOW |
| ADD COLUMN（NOT NULL，無 default） | **HIGH** |
| ALTER COLUMN（narrowing 類型） | **HIGH** |
| DROP COLUMN / DROP TABLE | **HIGH** |
| CREATE TABLE | LOW |

**硬規則：**
- 任何 migration 文件必須在 Approval 後才生成
- 無 down migration 不得輸出 up migration
- HIGH RISK 需明確輸入 `APPROVE HIGH RISK`（"yes"/"ok" 不算）
- 不可跳過任何 section

**驗證摘要：**
- Trigger 衝突風險：極低（DB 術語語義獨特）
- 自由度：最嚴格 Narrow Bridge（7 段強制、版面固定）
- Failure Mode：資訊不足只問一個問題，缺失資訊不阻斷 Approval Gate

---

### 4. auditing-dependencies

```yaml
---
name: auditing-dependencies
description: >
  Performs a seven-point health audit on any external package before it enters
  the codebase, preventing hallucinated packages and dependency bloat. Invoke
  this skill when a user says "install", "npm install", "add dependency",
  "import", "new package", or "pip install". It checks existence, redundancy
  against current dependencies, maintenance activity, popularity, license
  compatibility, bundle size, and known CVEs, then delivers a PASS, WARN, or
  BLOCK verdict with a replacement recommendation when blocked.
---
```

**七項審查（全部執行，不可跳過）：**

| # | 檢查 | BLOCK 條件 |
|---|------|-----------|
| 1 | Existence | 套件不存在 → BLOCK（幻覺風險） |
| 2 | Redundancy | 已有等效套件 → BLOCK |
| 3 | Activity | Repository 已封存 → BLOCK |
| 4 | Popularity | < 10k/week 下載 → WARN |
| 5 | License | 無授權/未知 → BLOCK；GPL/AGPL → WARN |
| 6 | Bundle Size | > 500 KB gzipped → BLOCK（backend 跳過） |
| 7 | Security | High/Critical CVE → BLOCK |

**Verdict 輸出格式：**
```
DEPENDENCY AUDIT REPORT
────────────────────────
Package:   <name>@<version>

  Check 1 — Existence:    PASS / WARN / BLOCK
  ...（七項）

FINAL VERDICT: PASS / WARN / BLOCK
────────────────────────
[一句建議]
```

**Verdict 聚合：** 任一 BLOCK → FINAL=BLOCK；有 WARN 無 BLOCK → FINAL=WARN

**硬規則：**
- BLOCK 必須提供具體替代方案
- 套件不存在 → 輸出 HALLUCINATION RISK 警告
- package.json 不可讀 → Check 2 標 SKIPPED，繼續其他 6 項

**驗證摘要：**
- Trigger 衝突風險：低；"import" 最寬，限定「外部套件」
- 自由度：Narrow Bridge（算法式 Verdict 聚合）
- Failure Mode：依賴清單缺失 → 跳過 Check 2，標記說明

---

### 5. reviewing-security

```yaml
---
name: reviewing-security
description: >
  Performs a comprehensive, multi-axis security audit of code before high-risk
  features ship to production. Invoke this skill when the user says "security
  review", "security audit", "review for security", "is this secure", or
  "before we ship" — especially when the code touches authentication,
  authorization, PII handling, payment flows, or file uploads. Unlike the
  Security axis inside auto-code-review (which is a single pass during general
  code review), this skill runs nine dedicated axes: Injection, Authentication,
  Authorization, Sensitive Data Exposure, Misconfiguration, CSRF/SSRF, Secrets
  Leakage, Input Validation, and Dependency CVEs. Each finding is rated on a
  four-level severity scale (CRITICAL / HIGH / MEDIUM / LOW) with an explicit
  ship-gate: CRITICAL findings block release; others are triaged by sprint.
---
```

**九個審查軸（全部執行，即使無問題也明確回報「無問題」）：**

| # | 軸 | 核心項目 |
|---|----|----|
| 1 | Injection | SQL/Command/LDAP/XSS |
| 2 | Authentication | bcrypt/argon2、Session、JWT alg 驗證 |
| 3 | Authorization | 水平越權（A 讀 B 的資料）、垂直越權 |
| 4 | Sensitive Data | PII logging、over-fetching、HTTPS |
| 5 | Misconfiguration | CORS、CSP/HSTS、error message 洩漏 |
| 6 | CSRF/SSRF | CSRF token、URL 白名單 |
| 7 | Secrets | Hardcoded creds、.gitignore 缺失 |
| 8 | Input Validation | Boundary 驗證覆蓋 |
| 9 | Dependencies | 已知 CVE |

**Severity Scale：**
- CRITICAL → 封鎖上線
- HIGH → 本 Sprint 修
- MEDIUM → 下個版本
- LOW → 技術債追蹤

**與 auto-code-review 的分工：**
> `auto-code-review` Axis 8 = 生成時的快速掃描（一軸）
> `reviewing-security` = 上線前九軸深度審查 + OWASP 修復指引

**驗證摘要：**
- Trigger 衝突風險：低（"security review/audit" 語義明確）
- Failure Mode：無程式碼 → 詢問「請提供待審查的程式碼」

---

### 6. writing-tests

```yaml
---
name: writing-tests
description: >
  Automatically generates tests for existing code by first reading the
  project's current test files to match their naming conventions, assertion
  style, and folder structure — never inventing a style from scratch. Invoke
  this skill when the user says "write tests", "add tests", "test coverage",
  "generate tests", or "missing tests", and when a new feature is complete,
  after a refactor, or when auto-code-review flags insufficient test coverage.
  Produces tests covering Happy Path, Boundary Cases (null / empty / max
  length / zero), and Error Paths. Each test description follows the pattern
  "should [outcome] when [condition]". Never mocks third-party dependencies
  you do not own; uses real implementations or official test doubles instead.
---
```

**核心行為（Step 0 強制先行）：**

0. **讀現有測試文件**，確認：框架、命名慣例、assertion style、目錄結構、mock 風格
1. 覆蓋三個 Category（缺一不可）：
   - **Happy Path**：正常輸入的預期輸出
   - **Boundary Cases**：null/empty/zero/max/whitespace/unicode
   - **Error Paths**：無效類型、依賴失敗、業務規則違反

**測試描述規則（強制）：**
```
should [expected outcome] when [specific condition]
✅ "should return 404 when user id does not exist"
❌ "test 1" / "works correctly"
```

**Mock 規則：**
- ✅ 可 mock：第三方外部 API、系統時鐘、隨機數
- ❌ 不可 mock：自家 internal module、語言 built-in、integration test 中的 DB

**驗證摘要：**
- 自由度：混合型（三 Category 結構強制，具體實作開放）
- Failure Mode：無程式碼 → 詢問模組名稱和原始碼

**子文件：** `docs/TEST_TEMPLATE.md`（Jest/pytest/Go/RSpec 各框架範本）

---

### 7. refactoring-safely

```yaml
---
name: refactoring-safely
description: >
  Captures a binding pre-refactor contract before any code is restructured.
  Use this skill whenever the user says "refactor", "clean up", "restructure",
  "reorganize", or "simplify the code". The skill snapshots every public
  function signature, exported type, and module consumer so that post-refactor
  behaviour can be verified against an immutable baseline. It then pauses and
  waits for explicit approval before any code is touched, preventing silent
  API breakage in downstream callers.
---
```

**四個 Phase（強制序列）：**

**Phase 1**（靜默）：讀目標文件、搜索全 codebase 的消費者

**Phase 2**（固定格式輸出）：
- **Public API Snapshot**：所有 exported symbol 的完整簽名
- **Consumer Map**：所有 import 此模組的文件清單
- **Invariants Declaration**：從 tests/docs 推導出的行為契約
- **Existing Test Coverage**：對應測試文件清單
- **Risk Register**：HIGH/MEDIUM/LOW 風險標記

**Phase 3**（Gate，輸出後停止）：
```
⚠️  PRE-REFACTOR SNAPSHOT COMPLETE
Type APPROVED to begin refactoring, or correct any item above.
```

**Phase 4**（APPROVED 後）：逐 unit 重構，每個 unit 後展示 diff + 說明哪些 invariants 保留

**硬規則：**
- Phase 3 確認前絕不修改任何文件
- Breaking change 需要第二次 APPROVED
- "just do it" → 拒絕並提示先確認 snapshot

**驗證摘要：**
- Trigger 衝突風險：低（"refactor" 語義聚焦）
- Failure Mode：範圍不明確 → 問一個問題，不盲目掃描

---

### 8. debugging-systematically

```yaml
---
name: debugging-systematically
description: >
  Runs a structured root-cause analysis protocol when a bug resists quick
  fixes. Invoke this skill when the user says "still broken", "tried three
  times", "I don't know why", "same error", or "still failing". The skill
  resets all prior assumptions, classifies the error type, traces data flow,
  proposes five ranked hypotheses with the fastest verification method for
  each, and enforces a no-repeat-fix rule: Claude may not attempt a second
  fix for the same hypothesis without verification evidence. Prevents the
  infinite guess-and-retry loop that wastes sessions and erodes trust.
---
```

**六個 Phase（強制序列）：**

**Phase 1 — Assumption Reset：** 完整清除假設，列出所有已嘗試方法及結果

**Phase 2 — Error Taxonomy：**
分類為 Syntax / Runtime / Logic / Environment / Concurrency / External

**Phase 3 — Data Flow Trace：**
從 entry 到 failure point 逐步追蹤，未知步驟標 `[UNKNOWN — needs: ...]`

**Phase 4 — 5 個假說排名（最關鍵）：**
```yaml
hypothesis-1:
  statement: [根因描述]
  probability: High/Medium/Low
  evidence-for: [支持證據]
  verification:
    command: [最快驗證命令]
    time-estimate: [< 1min]
  fix-if-confirmed: [概述，不寫程式碼]
```
輸出後，**VERIFICATION CHECKPOINT → 停止等待驗證結果**

**Phase 5**：用戶回報結果後更新 excluded 清單，不重複測試已排除的假說

**Phase 6**：假說確認後，輸出最小化 Targeted Fix（移除任一行就不夠用）

**硬規則：**
- 同一假說不得連續嘗試 2 次 Fix 未得到驗證
- 維護 excluded 假說清單，絕不回頭
- "just guess" → 拒絕，指定驗證方法

**驗證摘要：**
- Trigger 語義：「已嘗試且失敗」的心理狀態，不與初次 debug 衝突
- Failure Mode：無錯誤訊息 → Phase 2/3 大量 UNKNOWN，引導用戶提供

---

### 9. summarizing-pr

```yaml
---
name: summarizing-pr
description: >
  Generates a standardized pull request description from the current
  diff and conversation context. Invoke this skill when the user says
  "write PR description", "summarize my changes", "create PR",
  "PR ready", or "write the PR". The skill produces a complete PR
  body with six fixed sections: What changed, Why, How to test,
  Risks and Edge Cases, Rollback instructions, and a pre-merge
  Checklist. Output is formatted as a GitHub-compatible Markdown
  block ready to paste. Eliminates vague commit messages and ensures
  reviewers have all context needed to approve safely.
---
```

**固定六段 PR Description（順序不可變）：**

1. **What changed** — 具體變更 bullet list（禁止 "improved"/"fixed stuff"）
2. **Why** — 動機段落 + issue 連結
3. **How to test** — 可重現的步驟 + 預期結果
4. **Risks & Edge Cases** — Markdown table，含 Severity 和 Mitigation
5. **Rollback** — 明確回滾命令 + 預估時間
6. **Checklist** — `- [ ]` 格式（Claude 不得預先勾選）

**Phase 3 品質自我檢查：**
輸出後自動執行 audit，若 NEEDS CLARIFICATION 則列出具體問題

**硬規則：**
- 禁止寫 "fix stuff"/"various improvements"
- Rollback section 永遠存在（即使是"小"改動）
- Checklist 項目永不預先勾選
- "write something quick" → 仍輸出完整六段，缺失資訊用 `[needs clarification]`

**驗證摘要：**
- 自由度：Narrow Bridge（六段結構固定，Risks 強制用 table）
- Failure Mode：無 diff → 六段全標 `[needs clarification]`，輸出 NEEDS CLARIFICATION

**子文件：** `PR_TEMPLATE.md`（手動填寫參考模板，含各段填寫指引）

---

### 10. running-retrospective

```yaml
---
name: running-retrospective
description: >
  Conducts a structured post-feature retrospective that transforms this
  session's AI errors and effective patterns into permanent CLAUDE.md rules.
  Invoke this skill when the user says "retro", "retrospective",
  "let's do a retro", "what did we learn", or "update CLAUDE.md".
  The skill extracts effective prompt patterns worth keeping, identifies
  AI misjudgement patterns that should become explicit prohibitions, produces
  a ready-to-apply CLAUDE.md diff, and recommends off-limits zone updates.
  Makes each CLAUDE.md incrementally more accurate with each completed feature.
---
```

**六個 Phase：**

**Phase 1**（靜默回顧）：讀完整對話，收集五類資料點（有效 prompts、失敗交互、修正模式、越界行為、重複指令）

**Phase 2**：有效 Prompt 模式（2-8 個，含可重用性評分）

**Phase 3**：AI 誤判紀錄（每個含 category + 具體 proposed-rule）
```
misjudgement-1:
  category: Scope-Creep / Wrong-Assumption / Over-Engineering / ...
  proposed-rule: "NEVER [具體禁止]"
  confidence: High / Medium
```

**Phase 4**：Ready-to-apply CLAUDE.md diff（先讀現有 CLAUDE.md 避免重複）

**Phase 5**：Off-limits zone 推薦（新增/現有/考慮移除）

**Phase 6**：Session Score + Next-session primer（2-3 句可貼給下個 Session 的 Claude）

**硬規則：**
- 每個 proposed-rule 必須有 evidence citation
- 禁止捏造未發生的誤判
- 規則必須 imperative（NEVER/ALWAYS 開頭）、不得模糊（"be more careful"）
- Medium confidence → 加入 CLAUDE.md 但以 comment 標記

**驗證摘要：**
- Trigger 語義：「回顧/學習」，與其他 skill 完全無交集
- Failure Mode：對話太短 → 誠實回報 0 個有效模式，不捏造

**子文件：** `RETRO_GUIDE.md`（如何應用 diff、誤判 category 說明、Good rule vs. Bad rule 範例）

---

## 三、跨技能驗證摘要表

| 技能 | Trigger 衝突風險 | 自由度 | 不可移除段落 |
|------|----------------|--------|------------|
| setting-up-context | 低 | Narrow | CLAUDE.md 缺失 Stop Block、Confirmation Protocol |
| contracting-api | 低 | Narrow | YAML template、Hard Rules |
| migrating-schema | 極低 | 最嚴格 Narrow | Risk 分類表、Approval Gate |
| auditing-dependencies | 低 | Narrow | 七項 Verdict rules、聚合規則 |
| reviewing-security | 低 | Narrow | 九軸定義、Output Format |
| writing-tests | 低 | Mixed | Step 0 讀取清單、Mocking Rules |
| refactoring-safely | 低 | Narrow | Phase 3 Gate block |
| debugging-systematically | 低 | Narrow | VERIFICATION CHECKPOINT、excluded 清單 |
| summarizing-pr | 低 | Narrow | 六段格式、Checklist 不勾選規則 |
| running-retrospective | 極低 | Mixed | evidence citation 強制要求 |

---

## 四、文件結構規劃

```
.claude/skills/
├── auto-code-review/          ← 現有，保留
│   ├── SKILL.md
│   ├── CHECKLIST.md
│   ├── OUTPUT_EXAMPLES.md
│   └── README.md
├── idea-refiner/              ← 現有，保留
│   ├── SKILL.md
│   ├── QUESTION_BANK.md
│   ├── OUTPUT_EXAMPLES.md
│   └── README.md
├── implementation-planner/    ← 現有，保留
│   ├── SKILL.md
│   ├── TASK_TEMPLATE.md
│   ├── OUTPUT_EXAMPLES.md
│   └── README.md
├── setting-up-context/        ← 新增
│   ├── SKILL.md
│   ├── SESSION_TEMPLATE.md
│   └── README.md
├── contracting-api/           ← 新增
│   ├── SKILL.md
│   └── README.md
├── migrating-schema/          ← 新增
│   ├── SKILL.md
│   └── README.md
├── auditing-dependencies/     ← 新增
│   ├── SKILL.md
│   └── README.md
├── reviewing-security/        ← 新增
│   ├── SKILL.md
│   ├── docs/SECURITY_CHECKLIST.md
│   └── README.md
├── writing-tests/             ← 新增
│   ├── SKILL.md
│   ├── docs/TEST_TEMPLATE.md
│   └── README.md
├── refactoring-safely/        ← 新增
│   ├── SKILL.md
│   └── README.md
├── debugging-systematically/  ← 新增
│   ├── SKILL.md
│   └── README.md
├── summarizing-pr/            ← 新增
│   ├── SKILL.md
│   ├── PR_TEMPLATE.md
│   └── README.md
└── running-retrospective/     ← 新增
    ├── SKILL.md
    ├── RETRO_GUIDE.md
    └── README.md
```

---

## 五、確認後的實作順序

建議分批實作，按風險/價值優先：

**Batch 1（最高優先，防禦性）：**
1. `setting-up-context` — 每個 session 的前置
2. `migrating-schema` — 防止 DB 資料遺失

**Batch 2（品質閘門）：**
3. `reviewing-security` — 上線安全把關
4. `auditing-dependencies` — 套件引入審查
5. `refactoring-safely` — 重構前置契約

**Batch 3（開發效率）：**
6. `writing-tests` — 測試補全
7. `debugging-systematically` — 卡住時根因分析
8. `contracting-api` — API 契約鎖定

**Batch 4（收尾工作流）：**
9. `summarizing-pr` — PR Description
10. `running-retrospective` — CLAUDE.md 持續改進

---

**確認此提案後請回覆，即開始按 Batch 1 順序實作。**
