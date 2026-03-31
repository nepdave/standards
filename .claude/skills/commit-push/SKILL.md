---
name: commit-push
description: Summarize code changes into a commit message, commit, and push to the current branch.
model: claude-haiku-4-5-20251001
allowed-tools: Bash
---

1. Run `git status` and `git diff --staged` and `git diff` to see all changes.
2. Run `git log --oneline -5` to match the repo's commit message style.
3. Stage all relevant changed files (be specific — do not use `git add -A`). Never stage secrets, credentials, or binaries.
4. Write a commit message with two parts. Do NOT include "Co-Authored-By" lines.
   - **Line 1:** A short summary of what was done (1 sentence, matches repo style)
   - **Lines 3+:** A bulleted list itemizing each change, one per line, prefixed with `-`
   Example:
   ```
   self-serve API key signup flow

   - add migration for email column on api_key table
   - add CreateKeyForEmail() with email validation
   - add SignupHandler and HTMX signup route
   - add signup page and success fragment templates
   - update nav and hero CTAs to point to /signup
   ```
5. Commit using a HEREDOC for the message:
   ```
   git commit -m "$(cat <<'EOF'
   your message here
   EOF
   )"
   ```
6. Push to the current branch: `git push origin HEAD`
7. Report the commit hash and branch name when done.
