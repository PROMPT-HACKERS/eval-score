# Ship: Commit, Push, and Create PR

Commit all changes, push to remote, and create a pull request in one workflow.
Handles both new PRs and adding commits to existing PRs.

## Workflow

### 1. Check current state

Run `git status` (without -uall flag) and `git diff` to understand what has changed. If there are no changes, inform the user and stop.

### 2. Analyze changes

Run `git diff` and `git diff --cached` to review all staged and unstaged changes. Understand the nature of each change (new feature, bug fix, refactoring, etc.).

### 3. Stage and commit

- Stage all relevant changed files individually by name (do NOT use `git add -A` or `git add .`). Never stage files that likely contain secrets (.env, credentials, etc.).
- Write a concise commit message in English that focuses on "why" rather than "what", following the project's Tidy First methodology.
- End the commit message with: `Co-Authored-By: Claude Opus 4.6 <noreply@anthropic.com>`
- Use a HEREDOC for the commit message to ensure correct formatting.

### 4. Determine branch and PR state

Check the current branch and whether a PR already exists:

```
git branch --show-current
gh pr view --json url,title 2>/dev/null
```

Based on the result, follow the appropriate path:

- **On `main`, no PR**: Create a new branch (`feat/`, `fix/`, or `refactor/`). Ask the user for the branch name if unclear.
- **On feature branch, no PR**: Use the current branch as-is. A new PR will be created in step 6.
- **On feature branch, PR exists**: Use the current branch as-is. Skip step 6 (PR creation) since pushing will update the existing PR.

### 5. Push to remote

Push the current branch to origin with the `-u` flag:
```
git push -u origin <branch-name>
```

### 6. Create Pull Request (only if no existing PR)

Skip this step if a PR already exists for the current branch.

Create a PR using `gh pr create`:

1. Read `.github/PULL_REQUEST_TEMPLATE` to get the project's PR template
2. Fill in each section of the template based on the changes being committed (content in Japanese)
3. Replace HTML comments (`<!-- ... -->`) with actual content
4. Append `🤖 Generated with [Claude Code](https://claude.com/claude-code)` at the end of the body
5. Pass the filled-in template as `--body` using a HEREDOC, with a concise `--title` (under 70 chars, in English)

### 7. Report result

Show the user a summary of what was done:
- If new PR: show the PR URL
- If existing PR: show the PR URL and note that the new commit was pushed to the existing PR

## Rules

- All commit messages and PR titles must be in English (per AGENTS.md)
- PR body content must be in Japanese
- Always read `.github/PULL_REQUEST_TEMPLATE` for the PR body structure; do not hardcode the template
- Never commit .env or credential files
- Never force push
- Never amend existing commits
- If any step fails, stop and report the error to the user
