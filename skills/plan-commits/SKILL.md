---
name: plan-commits
description: Analyze all uncommitted changes in a git repo and organize them into logical, granular commits. Use this skill when the user wants to split their work into multiple commits, organize messy changes, plan a commit strategy, clean up before pushing, or mentions "plan commits", "split commits", "organize commits", "commit plan", or has a large set of uncommitted changes they want to break apart. Also trigger when the user says things like "help me commit this", "I need to make several commits from these changes", or "break these changes into commits".
---

# Plan Commits

Analyze all uncommitted changes in a git repository and organize them into logical, granular commits. Present the plan to the user, incorporate their feedback, and execute the commits on approval.

## Why this skill exists

Developers often implement multiple features, fixes, or refactors in a single working session and end up with a tangled set of changes. Clean, atomic commits make history readable, bisectable, and reviewable. This skill automates the tedious work of analyzing changes, figuring out logical groupings, and carefully staging partial files.

## Phase 1: Analyze all changes

Run these commands in parallel to build a complete picture of the working tree:

```
git status --short
git diff                          # unstaged changes to tracked files
git diff --cached                 # staged changes
git log --oneline -10             # recent commit style reference
```

Then, for every changed or new file, read the actual content to understand *what* each change does — not just *which* files changed. For modified files, read the diff. For new (untracked) files, read the full file. Group your reading into parallel batches for speed.

Pay attention to:
- What **domain concept** each change belongs to (auth, models, config, docs, tests, etc.)
- Whether a single file contains changes for **multiple logical units** (e.g., a settings file that adds both a new dependency config and a JWT config)
- **Dependencies between changes** — a model must exist before its endpoints can work
- The project's existing **commit message style** from `git log`

## Phase 2: Group into commits

Organize changes into the most granular logical commits possible, following these principles:

### Ordering
- Infrastructure and config changes first (dependencies, settings)
- Data layer next (models, migrations, signals)
- Business logic after that (serializers, views, endpoints)
- Tests alongside or right after the code they test
- Documentation last

### Granularity guidelines
- **Separate concerns**: a new dependency and its configuration is one commit; a new model is another
- **Keep related things together**: a model + its migration + its tests can be one commit if they form a single logical unit
- **Split shared files**: if `settings.py` has changes for two unrelated features, split them across commits using selective staging
- **Tests travel with code**: if a commit adds a feature, include the tests for that feature in the same commit — don't batch all tests into a separate commit unless they are purely test-infrastructure changes
- **Docs can be grouped**: README updates, task tracking files, and roadmap changes can go in a single "docs" commit unless they are tightly coupled to a specific feature commit

### Handling files that span multiple commits

When a single file has changes that belong to different commits, you have two options:

1. **Selective line staging with `git add -p`** — Use this when the changes are in clearly separable hunks. Feed responses to `git add -p` non-interactively using `printf` piped into the command, or use `git add -p` with the `--no-interactive` approach by creating temporary patch files.

2. **Staged backup approach** — For complex cases:
   - Save the full final state of the file
   - Apply only the changes needed for the current commit
   - Commit
   - Restore the full state for the next commit

   In practice, for most cases you can use `git diff` to create patch files and apply them selectively.

3. **Accept coarser grouping** — If splitting a file would be fragile or error-prone, it's better to group the changes together in one commit than to risk breaking the working tree. Mention this trade-off in the plan.

## Phase 3: Present the plan

Show the user a numbered plan in this format:

```
## Commit Plan (N commits)

### Commit 1: `<type>(<scope>): <concise message>`
**Files:**
- `path/to/file.py` — description of what changes
- `path/to/other.py` (partial) — only the X-related changes

> Brief explanation of what this commit accomplishes.

### Commit 2: `<type>: <concise message>`
...

---

**Notes:**
- Any trade-offs, files that couldn't be cleanly split, or ordering considerations
```

### Commit message conventions — Conventional Commits (required)

Always use the [Conventional Commits](https://www.conventionalcommits.org/) specification. This is not a fallback — it is the standard regardless of what `git log` shows.

**Format:**
```
<type>(<optional scope>): <description>
```

**Types** (use exactly these):
| Type | When to use |
|---|---|
| `feat` | A new feature or capability |
| `fix` | A bug fix |
| `docs` | Documentation only (READMEs, task files, roadmaps, comments) |
| `chore` | Dependency updates, config, tooling, CI — no production logic change |
| `refactor` | Code restructuring without behavior change |
| `test` | Adding or updating tests only (when tests are committed separately from the code they test) |
| `style` | Formatting, whitespace, linting fixes — no logic change |
| `perf` | Performance improvement |
| `ci` | CI/CD pipeline changes |
| `build` | Build system or external dependency changes |
| `revert` | Reverting a previous commit |

**Scope** (optional but encouraged): a short noun describing the area of the codebase, e.g. `auth`, `notes`, `api`, `ui`, `config`. Use lowercase, no spaces. Derive the scope from the domain concept the commit touches.

**Rules:**
- Description starts with a lowercase verb in imperative mood ("add", "fix", "update" — not "added", "fixes", "updates")
- Keep the full subject line under 72 characters
- No period at the end of the description
- Add the co-author trailer as required by the system
- If a commit includes both a feature and its tests, use `feat` — `test` is only for standalone test changes
- If a commit includes both config and a feature that needs that config, use `feat` with a scope

**Examples:**
```
feat(auth): implement signup endpoint with JWT token generation
fix(notes): prevent duplicate category names per user
chore: add drf-spectacular dependency and OpenAPI config
docs: update backend README with API endpoint reference
refactor(notes): extract category validation into serializer
test(auth): add integration tests for token refresh flow
chore(config): configure SIMPLE_JWT token lifetimes
feat(notes): add category CRUD endpoints with note count annotation
```

After presenting the plan, explicitly ask:

> "Does this plan look good? You can ask me to merge or split commits, reorder them, change messages, or adjust groupings. Say **go** when you're ready to execute."

## Phase 4: Incorporate feedback

The user might ask to:
- **Merge** commits ("combine 2 and 3")
- **Split** a commit further ("separate the migration from the model")
- **Reorder** commits
- **Change a commit message**
- **Move a file** from one commit to another

Update the plan and present it again. Repeat until the user approves.

## Phase 5: Execute commits

Once approved, execute each commit sequentially:

1. For each commit in order:
   a. Stage exactly the files (or partial files) listed in the plan
   b. Verify what's staged with `git diff --cached --stat`
   c. Create the commit with the agreed message, using a HEREDOC for proper formatting:
      ```bash
      git commit -m "$(cat <<'EOF'
      feat(auth): implement signup endpoint

      Co-Authored-By: Claude <co-author-info>
      EOF
      )"
      ```
   d. Run `git status --short` to confirm the commit succeeded and the working tree is in the expected state

2. After all commits are done, show a final summary:
   ```
   git log --oneline -N   # where N = number of commits just created
   ```

### Error handling
- If a commit fails (e.g., pre-commit hook), fix the issue, re-stage, and create a NEW commit — never amend
- If staging a partial file goes wrong, `git checkout -- <file>` to restore and try again
- If something unexpected happens, stop and tell the user rather than guessing

## Important rules

- **Never amend existing commits** — only create new ones
- **Never force push** or run destructive git commands
- **Never skip hooks** (`--no-verify`)
- **Never stage sensitive files** (`.env`, credentials, secrets) — warn the user if they appear
- **Verify each step** — check `git diff --cached` before committing, check `git status` after
- **Preserve the working tree** — after all commits, the repo should be in a clean state with no unintended changes lost
