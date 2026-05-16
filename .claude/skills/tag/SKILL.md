---
name: tag
description: Tag the current commit to mark a chapter milestone
---

# Tag Skill

When the user invokes `/tag <chapter-number> <name>`, create a lightweight
git tag to mark the completion of that chapter.

## Tag Naming

```
ch<XX>-<name>
```

Examples: `ch04-scanner`, `ch12-classes`, `ch26-gc`

## Workflow

### Step 1 — Check for uncommitted changes

```bash
git status --porcelain
```

### Step 2 — Commit if needed

If there are uncommitted changes (staged or unstaged), invoke the `commit`
skill to create a commit first. Do NOT tag with a dirty working tree.

### Step 3 — Create the tag

```bash
git tag ch<XX>-<name>
```

Always use lightweight tags (no `-a`, no `-m`). The commit message already
carries the context; the tag is just a chapter bookmark.

### Step 4 — Confirm

Show the tag that was created:
```bash
git tag -l "ch*" | sort
```

**Do NOT push.**
