---
name: commit
description: Create a formatted git commit following conventional commit style
---

# Commit Skill

When the user invokes `/commit`, follow this workflow to create a properly formatted git commit.

## Commit Message Format

```
type(scope): short description (≤72 chars)

Multi-line body explaining what changed and why. Each line ≤72 chars.
Reference relevant chapters, design decisions, or trade-offs.
```

### Types
`feat` `fix` `chore` `refactor` `docs` `test` `style` `perf`

### Scopes
Module names that map to the project structure:

| Scope | Crate/Area |
|-------|-----------|
| `scanner` | Lexer / tokenization |
| `parser` | Recursive descent parser |
| `interpreter` | Tree-walk evaluator (rlox-walk) |
| `resolver` | Variable resolver |
| `compiler` | Bytecode compiler (rlox-byte) |
| `vm` | Bytecode VM |
| `gc` | Garbage collector |
| `table` | Hash table |
| `value` | Value representation |
| `object` | Obj / heap objects |
| `all` | Cross-cutting changes |

When a change spans multiple modules, pick the most relevant one. Use `all` only for truly cross-cutting changes (e.g. workspace Cargo.toml, CI, repo-level docs).

## Workflow

### Step 1 — Gather information
Run these commands in parallel to understand the current state:

```bash
git status
git diff --staged
git diff
git log --oneline -10
```

### Step 2 — Analyze changes
Identify:
- Which crate(s) are affected (rlox-walk, rlox-byte, or both)
- What kind of change it is (new feature, bug fix, refactor, etc.)
- Which module is primarily affected
- Whether the changes contain **unrelated groups** that should be separate commits

**Split decision rule**: Use the `AskUserQuestion` tool to ask the user whether to split into multiple commits — but **only when the changes contain two or more clearly unrelated topics**. Examples of unrelated topics: a new feature in scanner PLUS a README rewrite; a bug fix in parser PLUS tooling config in .claude/. In such cases, list the distinct change groups as options so the user can choose "split" or "single commit".

When all changes are coherent (e.g. all touching one module for one purpose), skip the split question and proceed directly to drafting.

### Step 3 — Draft the commit message
Write the commit message in English. Follow these rules:

1. **Subject line**: `type(scope): short description` — max 72 chars, lowercase, no period at end
2. **Blank line** after subject
3. **Body**: Explain WHAT changed and WHY. Each line ≤72 chars. Can be multiple paragraphs separated by blank lines.

### Step 4 — Confirm with user
Use the `AskUserQuestion` tool to present the drafted commit message and ask the user to confirm. Single-select: "Approve" or "Edit". If the user chooses "Edit", ask what they want changed, revise, and re-confirm.

### Step 5 — Execute
Once confirmed:
```bash
git add <relevant files>  # Only the changed files, never -A or .
git commit -m "<message>"
```

**Do NOT push.** The user will push manually when ready.

## Examples

Good:
```
feat(scanner): add support for C-style block comments

Block comments (/* ... */) can now span multiple lines and nest.
Handles the edge case where an unclosed block comment reaches
EOF by reporting an error with the opening line number.
```

Good:
```
fix(parser): handle empty grouping expressions

The parser previously crashed on empty parentheses `()` because
it tried to consume an expression that didn't exist. Now it
reports "Expected expression" and synchronizes to the next token.
```

Bad:
```
added scanner stuff
```

Bad:
```
feat: scanner block comments
(no scope, no body explaining decisions)
```
