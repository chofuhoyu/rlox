---
name: tag
description: Tag the current commit to mark a chapter milestone
---

# Tag Skill

When the user invokes `/tag`, create a lightweight git tag to mark chapter
completion. **Derive the chapter from conversation context** — do not ask the
user to provide it unless it is genuinely ambiguous.

## Tag Naming

```
ch<XX>-<name>
```

Examples: `ch04-scanner`, `ch12-classes`, `ch26-gc`

## How to derive the chapter

Look at:

1. **Conversation context** — what chapter has the user been discussing?
   What code did they just ask to implement? What chapter mapping (from README)
   does that implementation belong to?
2. **Recent commits** — `git log --oneline -5` — do commit messages mention
   chapter numbers, module names, or implementation topics that map to a
   specific chapter?
3. **Changed files** — which crate and which module files? Map them to
   chapters via the README chapter table:

   | Chapter | Topic | Crate |
   |---------|-------|-------|
   | 4 | scanner | rlox-walk |
   | 5 | AST (Expr/Stmt) | rlox-walk |
   | 6 | parser | rlox-walk |
   | 7 | expression eval | rlox-walk |
   | 8 | statements | rlox-walk |
   | 9 | control flow | rlox-walk |
   | 10 | functions | rlox-walk |
   | 11 | resolver | rlox-walk |
   | 12 | classes | rlox-walk |
   | 13 | inheritance | rlox-walk |
   | 14 | chunk / bytecode | rlox-byte |
   | 15 | VM | rlox-byte |
   | 16 | scanner (on-demand) | rlox-byte |
   | 17 | expression compile | rlox-byte |
   | 18 | value types | rlox-byte |
   | 19 | strings | rlox-byte |
   | 20 | hash table | rlox-byte |
   | 21 | global variables | rlox-byte |
   | 22 | local variables | rlox-byte |
   | 23 | jumps | rlox-byte |
   | 24 | function calls | rlox-byte |
   | 25 | closures | rlox-byte |
   | 26 | GC | rlox-byte |
   | 27 | class instances | rlox-byte |
   | 28 | methods | rlox-byte |
   | 29 | superclasses | rlox-byte |
   | 30 | optimization | rlox-byte |

4. **Only if ambiguous** — use `AskUserQuestion` to ask: "Which chapter does
   this tag mark?" with a few likely options.

## Workflow

### Step 1 — Check for uncommitted changes

```bash
git status --porcelain
```

### Step 2 — Commit if needed

If there are uncommitted changes, invoke the `commit` skill first.
Do NOT tag with a dirty working tree.

### Step 3 — Derive chapter and create tag

From context, determine `<XX>` and `<name>`, then:
```bash
git tag ch<XX>-<name>
```

Always lightweight (no `-a`, no `-m`).

### Step 4 — Confirm

```bash
git tag -l "ch*" | sort -V
```

**Do NOT push.**
