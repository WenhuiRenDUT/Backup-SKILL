---
name: ai-branch-draft-pr
description: Workflow for coding agents in Antigravity or pre-branched environments. Automatically detects the current task branch, opens a GitHub Draft PR at turn 1, creates incremental wip commits on every turn, supports in-place file reversion from main, finalizes to formal PR upon user approval, and automatically syncs local code after user confirms merge. Use when writing code, modifying numerical algorithms, or debugging in a Git repository.
---

# AI Branch & Draft PR Lifecycle Workflow

Standard operational procedure for autonomous coding agents operating in environments where branches and worktrees are pre-allocated by the IDE/platform (such as Antigravity). Ensures zero interference with the main branch during development, strict file scope protection, live web review via GitHub Draft PRs, in-place file rollbacks, and automatic post-merge local synchronization.

## When to Use

- When tasked with developing new solvers, algorithms, or benchmark scripts.
- When refactoring, debugging, or optimizing existing code in a Git repository.
- Whenever working across multi-turn iterations with human review via GitHub.

## Core Scope & Environmental Invariants

1. **Pre-allocated Branch:** Assume the IDE/user has already created and switched to the dedicated task branch. **NEVER execute `git checkout -b` or switch to `main` during task development.** (Switching to `main` is strictly reserved for Step 5 after the user confirms merge).
2. **Main Protection:** The `main` branch must never be touched directly during task development.
3. **Asset Protection:** NEVER modify `.gitignore`, repository root configs, CI workflows, or academic paper files (e.g., `paper/`, `*.tex`, `*.bib`).
4. **Remote Main as Reference:** All diffs, inspections, and file restorations must reference `origin/main` directly without checking out the branch.

---

## Execution Steps

### Step 1: Branch Verification & Initial Draft PR (Turn 1)

At the very beginning of a new task:

1. Detect and verify the current working branch:
  ```bash
  BRANCH=$(git branch --show-current)
  if [ "$BRANCH" = "main" ] || [ -z "$BRANCH" ]; then
    echo "Error: Operating directly on main or in a detached state is strictly forbidden!" && exit 1
  fi
  ```

2. Implement the initial scaffold or first code increment.

3. Stage intended code files and make the initial commit:
  ```bash
  git status
  git add <modified-code-files>
  git commit -m "feat(<scope>): initial implementation of <task>"
  ```

4. Push to remote and open a GitHub Draft PR:
  ```bash
  git push -u origin "$BRANCH"
  gh pr create --draft --base main \
    --title "[WIP] <type>(<scope>): <Task Title>" \
    --body "### Task Context
- Working branch: \`$BRANCH\`
- Real-time review available in the **Files changed** tab.

### Initial Scope
- Added initial implementation for <task>."
  ```

5. Output the PR URL so the user can open it in their browser for live monitoring.

---

### Step 2: Incremental Turn Commits (Turns 2 to N)

At the end of every conversation turn with code modifications:

1. Run local syntax checks or test cases to ensure the modifications execute without runtime errors.

2. Stage only the modified code files:
  ```bash
  git add <modified-code-files>
  git commit -m "wip(<scope>): <concise summary of this turn's change>"
  ```

3. Push immediately to update the Draft PR:
  ```bash
  git push origin HEAD
  ```

4. Notify the user that changes are pushed and ready for an `F5` refresh on GitHub.

---

### Step 3: In-Place File Reversion (During Review)

If the user reviews the PR on GitHub and requests discarding changes to specific files (e.g., unwanted config changes or unintended file edits):

1. **DO NOT switch branches.** Restore the file directly from `origin/main`:
  ```bash
  git checkout origin/main -- <path-to-unwanted-file>
  ```

2. If untracked garbage files were created, remove them:
  ```bash
  rm -rf <untracked-file-or-dir>
  ```

3. Commit the reversion and push:
  ```bash
  git commit -m "revert: restore <path-to-unwanted-file> from main"
  git push origin HEAD
  ```

4. Notify the user to refresh the PR page to verify that the file has disappeared from the diff.

---

### Step 4: Finalization & Formal PR Review Handoff

When the user explicitly permits or approves submitting a formal PR (例如：“可以提正式 PR 了”、“代码没问题，准备合并”):

1. Verify that only legitimate files are included in the cumulative diff:
  ```bash
  git diff --name-only origin/main...HEAD
  ```

2. Mark the Draft PR as ready for review:
  ```bash
  gh pr ready
  ```

3. Update the PR title by removing the `[WIP]` prefix:
  ```bash
  gh pr edit --title "<type>(<scope>): <Task Title>"
  ```

4. Notify the user to review and merge:
  - Inform the user that the PR has been updated to formal status and is ready for review (GitHub CLI / IDE 会自动关联展示 PR 状态与链接，无需额外抓取链接)。
  - Remind the user: 确认无误后在 GitHub 网页端点击 **Squash and merge**（并可删除分支）。
  - Prompt the user: "完成合并后请在对话中回复我已合并，我将自动同步本地代码到最新。"

---

### Step 5: Post-Merge Local Code Sync

When the user confirms the PR has been merged (例如：“我已经合并了”、“已合并”、“merged”):

1. Switch back to the `main` branch:
  ```bash
  git checkout main
  ```

2. Pull latest changes from remote to update local code to the latest:
  ```bash
  git pull origin main
  ```

3. Optionally clean up remote-tracking references and local task branch:
  ```bash
  git fetch --prune
  git branch -d "$BRANCH" 2>/dev/null || true
  ```

4. Verify and notify the user:
  ```bash
  git log -1 --oneline
  ```
  Output the latest commit hash and message, confirming that local code is now fully synced with `main`.

---

## Gotchas

- **Branch Discipline:** Do not switch branches during task development. Switching to `main` is strictly reserved for Step 5 after user explicit merge confirmation.
- **Triple-dot Diffing:** Always use `git diff --name-only origin/main...HEAD`. Two-dot diffing (`origin/main`) introduces false positives if remote `main` has progressed.
- **Selective Staging:** Never run a blind `git add .` without checking `git status` first, preventing unintended temporary artifacts, data caches, or paper files from entering the PR.
