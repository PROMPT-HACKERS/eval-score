---
name: review-diff
description: Review uncommitted git changes (staged and unstaged) in the current repository. Use when the user asks to review local changes, uncommitted diffs, working tree modifications, or wants feedback on code before committing. Triggers on phrases like "review diff", "review changes", "review my changes", "check my code", "レビューして", "差分レビュー", "コミット前レビュー", "変更をチェック", "レビュー".
---

# Review Diff

Review uncommitted git diffs following the project's AGENTS.md conventions and review.instructions.md guidelines.

## Workflow

### 1. Collect diff

Run `git diff` (unstaged) and `git diff --cached` (staged) to gather all uncommitted changes. If both are empty, inform the user there are no changes to review.

### 2. Read project rules

Read the following files for project-specific review criteria:
- `AGENTS.md` — Tidy First methodology, layer responsibilities, code quality standards
- `.github/instructions/review.instructions.md` — Label rules, language rules, feedback principles

### 3. Analyze changes

Review the diff against these criteria:

**Tidy First methodology:**
- Structural and behavioral changes must NOT be mixed in the same diff
- Structural changes should come before behavioral changes

**Layer responsibilities:**
- Handler layer: thin (20-30 lines), HTTP only, inline error handling, no business logic
- Usecase layer: thick (50-100 lines), all business logic, DB operations, transactions
- Extension layer: pure utilities, no DB access, no side effects

**Code quality:**
- No duplication
- Clear naming expressing intent
- Explicit dependencies
- Small, single-responsibility methods
- Minimal state and side effects
- Simplest solution that works

**Architecture rules:**
- No Repository layer abstractions
- No custom Input/Output structures when OpenAPI generates them
- No separate error handling functions in handlers
- Extension layer must not access database

### 4. Output review

Format output in Japanese following review.instructions.md label rules:

```
## レビュー結果

### [must] セキュリティ脆弱性、バグ、設計ルール違反
- `path/to/file:line` — 具体的な指摘と改善案

### [imo] 代替案、実装方法の提案
- `path/to/file:line` — 具体的な指摘と改善案

### [nits] 軽微な修正
- `path/to/file:line` — 具体的な指摘と改善案

### [ask] 質問・確認
- `path/to/file:line` — 意図の確認

### [fyi] 参考情報
- 背景説明、関連情報

### 総評
変更全体のサマリーと評価
```

**Label definitions:**
| Label | Usage |
|-------|-------|
| [must] | Must fix: security vulnerabilities, bugs, design rule violations |
| [imo] | Optional: alternative approaches, implementation suggestions |
| [nits] | Minor: typos, formatting, indentation |
| [ask] | Questions: intent clarification, decisions needed |
| [fyi] | FYI: background info, related context, best practices |

**Feedback principles:**
- Be specific — state exactly what to improve and how
- Be constructive — provide improvement suggestions, not just criticism
- Be respectful — use polite language
- Use Japanese for comments, English for project-specific terms (Usecase, Handler, etc.)
- Omit label sections that have no items
