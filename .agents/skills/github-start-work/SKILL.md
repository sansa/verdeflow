---
name: github-start-work
description: Find or create a VerdeFlow GitHub issue and, when repository files will change, create an issue-linked branch. Use only when asked to start tracked work, file an issue, or create a branch; ordinary edits do not trigger this workflow.
---

# Start tracked VerdeFlow work

Read the [GitHub work-start convention](../../../doc/contributing/GITHUB_WORKFLOW.md) and apply it to the current request. It is the shared source of truth for issue bodies, existing labels, risk descriptions, and branch names.

1. Confirm the affected repository and whether its files will change. Search open issues and reuse a matching issue.
2. Create or update the issue when requested. Use only labels that exist in that repository. If GitHub is unavailable, draft the body and report the missing write.
3. Create a branch from the latest `origin/main` only when repository files will change and an issue number is known. Preserve any uncommitted work; use a worktree if needed. Publish the branch with an upstream when the request calls for a GitHub branch.
4. Report the issue URL, labels, branch or reason for no branch, risk, and next step.

Use the GitHub connector, GitHub CLI, or another available GitHub interface. The convention does not grant permission to merge, close issues, push `main`, or alter other repositories.
