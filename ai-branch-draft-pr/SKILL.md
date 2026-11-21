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