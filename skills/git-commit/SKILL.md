---
name: git-commit
description: Creates git commits using Conventional Commits format. Use when committing changes to ensure consistent, meaningful commit messages.
allowed-tools: Bash, AskUserQuestion
user-invocable: true
---

# Git Commit Skill

Creates git commits following the [Conventional Commits](https://www.conventionalcommits.org/) specification. This ensures consistent, meaningful commit messages across the project.

## When to Use

Activate this skill when:
- User asks to commit changes
- User says "commit", "create commit", or "git commit"
- Changes are staged or ready to be committed
- At the end of a session when committing work

**This skill replaces the default commit workflow.**

## Conventional Commits Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Types

| Type | Description | Example |
|------|-------------|---------|
| `feat` | New feature or functionality | `feat: add user authentication` |
| `fix` | Bug fix | `fix: resolve login redirect issue` |
| `docs` | Documentation only | `docs: update README` |
| `style` | Code style (formatting, semicolons) | `style: fix indentation` |
| `refactor` | Code change that neither fixes nor adds | `refactor: extract validation logic` |
| `perf` | Performance improvement | `perf: optimize database queries` |
| `test` | Adding or fixing tests | `test: add user service tests` |
| `build` | Build system or dependencies | `build: update webpack config` |
| `ci` | CI configuration | `ci: add GitHub Actions workflow` |
| `chore` | Other changes (tooling, etc.) | `chore: update .gitignore` |

### Scope (Optional)

Scope indicates the module or directory affected. Use the most specific relevant name:
- `feat(auth): add OAuth support`
- `fix(session-start): correct template path`
- `refactor(skills): extract shared config parsing`

For changes spanning multiple modules, use the parent directory or omit scope:
- `refactor(skills): standardize config loading` (multiple skills changed)
- `chore: update dependencies` (project-wide)

### Breaking Changes

For breaking changes, add an exclamation mark after type/scope:

```
feat!: change API response format
refactor(auth)!: rename login endpoint
```

Or add a BREAKING CHANGE line in the footer.

## Instructions

### Step 1: Check Repository Status

Run:
```bash
git status
```

**If no changes:**
- Display: "No changes to commit."
- Stop skill execution

**If untracked files or modifications exist:**
- Continue to next step

### Step 2: Review Changes

Run in parallel:
```bash
git diff --staged
git diff
git log --oneline -5
```

Analyze:
- What files are changed/added
- Nature of changes (new feature, bug fix, refactor, etc.)
- Recent commit style for consistency

### Step 3: Stage Changes (if needed)

**If changes not staged:**
- Use AskUserQuestion:
  ```
  Found unstaged changes. What should I stage?
  ```
  Options:
  1. **Stage all changes** - `git add -A`
  2. **Stage specific files** - Ask which files
  3. **Only commit staged** - Proceed with currently staged only

### Step 4: Determine Commit Type

Based on the changes, suggest a commit type:

**Auto-detection hints:**
- New files with significant code -> `feat`
- Modified existing behavior -> Could be `fix`, `refactor`, or `feat`
- Only `.md` files -> `docs`
- Only formatting changes -> `style`
- Dependency changes -> `build`
- Test files only -> `test`

Use AskUserQuestion to confirm:
```
Based on the changes, I suggest:
  Type: [suggested-type]
  Scope: [suggested-scope or "none"]

Is this correct?
```

Options:
1. **Yes, use suggested** - Proceed with suggestion
2. **Change type** - Show type options
3. **Change scope** - Ask for scope
4. **Change both** - Ask for type and scope

### Step 5: Generate Description

Create a concise description (imperative mood, no period).

**Always use imperative mood — write it as a command: "add", "fix", "extract", not "added", "fixes", "extracting".**

**Describe WHAT changed and WHY — never HOW. The code shows how.**

- **Good**: "delegate commit responsibility to git-commit skill" (what + why, imperative)
- **Good**: "add optional implementation review step" (what, imperative)
- **Bad**: "move Steps 8-9 into a separate function and call it" (how)
- **Bad**: "Added authentication." (past tense, period)
- **Bad**: "adds auth" (present tense, too vague)
- **Bad**: "extracting validation logic" (gerund, not imperative)

Rules:
- Start with lowercase (unless proper noun)
- No period at end
- The full first line (`type(scope): description`) MUST be ≤ 80 characters
- If over 80, shorten the description — move detail to the body

### Step 6: Determine if Body Needed

**Add body if:**
- Changes are complex and need explanation
- Multiple related changes in one commit
- Breaking change needs context
- "Why" isn't obvious from description

**Skip body if:**
- Simple, self-explanatory change
- Single-purpose commit

If body needed, write 1-3 sentences explaining:
- **Why** this change was made
- **What** problem it solves
- Any important context

Do NOT describe HOW the code works — that belongs in the code itself.

### Step 7: Validate and Execute Commit

**Before committing, verify the first line is ≤ 80 characters.** If over 80, shorten the description and move detail to the body.

Build the commit message:

```bash
git commit -m "$(cat <<'EOF'
<type>[scope]: <description>

[body if needed]

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

**Always include the Co-Authored-By trailer.**

### Step 8: Verify Commit

Run:
```bash
git log -1 --format=full
git status
```

Display:
```
Commit created successfully!

[commit-hash] <type>[scope]: <description>

Files changed: [N]
Insertions: [+N]
Deletions: [-N]
```

---

## Examples

### Example 1: Simple Feature

**Changes**: New authentication middleware added

```
feat(auth): add authentication middleware

Implements JWT-based authentication for API endpoints.

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Example 2: Bug Fix

**Changes**: Fixed login redirect issue

```
fix(auth): correct login redirect URL

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Example 3: Documentation

**Changes**: Updated README

```
docs: update installation instructions

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Example 4: Refactor with Scope

**Changes**: Moved validation logic to separate module

```
refactor(utils): extract validation to separate module

Improves code organization by separating validation logic.
No functional changes.

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Example 5: Breaking Change

**Changes**: Renamed public API endpoint

```
feat(api)!: rename /users endpoint to /accounts

BREAKING CHANGE: /users endpoint removed. Use /accounts instead.
Migration: Update all API calls from /users to /accounts.

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Example 6: Chore

**Changes**: Updated dependencies

```
chore(deps): update dependencies to latest versions

Co-Authored-By: Claude <noreply@anthropic.com>
```

---

## Type Selection Guide

**Is it a new capability the user can use?** -> `feat`
**Does it fix broken behavior?** -> `fix`
**Is it restructuring without behavior change?** -> `refactor`
**Is it only documentation/comments?** -> `docs`
**Is it only whitespace/formatting?** -> `style`
**Does it make things faster?** -> `perf`
**Is it test code?** -> `test`
**Is it dependencies or build config?** -> `build`
**Is it CI/CD pipeline?** -> `ci`
**None of the above?** -> `chore`

---

## Notes

- Always include Co-Authored-By trailer for Claude contributions
- Use HEREDOC syntax for multi-line commit messages
- Verify commit succeeded before reporting success
- Don't amend commits unless explicitly asked
- Don't push unless explicitly asked (separate concern)
