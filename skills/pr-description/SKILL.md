---
name: pr-description
description: 'Write clear, concise, reviewer-friendly pull request descriptions. Use when drafting or improving a PR description, writing a PR summary, or preparing changes for code review. Focuses on WHAT the PR delivers and its impact, not line-by-line implementation details.'
argument-hint: 'Optional: PR title, branch name, or paste the diff/summary of changes'
---

# Pull Request Description Writer

Generate clear, concise PR descriptions that help reviewers understand what a change accomplishes without overwhelming them.

## When to Use
- Drafting a new pull request description
- Cleaning up or tightening an existing PR description
- Preparing a change set for code review
- Updating the description of an existing remote PR

## Core Principles
- Describe **WHAT** this PR delivers, not the implementation details or how it works.
- Keep it high-level. Reviewers can examine the diff for technical specifics.
- Assume reviewers are intelligent and busy — respect their time.
- Only surface file names, functions, or code details when there's a specific reason reviewers should scrutinize them (complex logic, edge cases, security concerns, architectural decisions).

## What to Include
- The problem being solved or feature being added.
- The user-facing or system-level impact.
- Key context: related issues, dependencies, breaking changes.
- Callouts for areas needing careful review (if applicable).

## What to Skip
- Line-by-line changes.
- Implementation approach or design patterns used.
- Obvious refactoring details.
- Generic function names or file paths (unless there's a reason to highlight them).

## Format
- The **first line** of the description must be the Jira ticket link (e.g. `https://<org>.atlassian.net/browse/ABC-123`), followed by a blank line, then the description body.
- Write a clean, scannable description — 2-4 sentences is often perfect.
- Use bullet points only when there are multiple distinct deliverables or important callouts.

## Procedure
1. Identify the change set: read the provided diff, branch, commits, or summary. If none is provided, inspect the current changes.
2. Find the Jira ticket link:
   - Infer it from the conversation context first (a pasted Jira URL, a ticket key like `ABC-123` mentioned by the user, the branch name, or commit messages). If only a key is found, build the full link from the Jira base URL seen elsewhere in the context.
   - If no ticket link or key can be found, **ask the user** for it before drafting. Do not invent one.
   - Put the link on the first line of the description.
3. Determine the single core outcome the PR delivers and its impact.
4. Draft the Jira link line, then 2-4 sentences capturing the problem/feature and impact.
5. Add bullets only for multiple deliverables or review callouts (breaking changes, security, edge cases).
6. Trim anything that restates the diff or explains implementation mechanics.

## Publish to the Remote PR
After drafting, update the description on the actual pull request. Determine the target PR, show the user the drafted description, then push it.

1. Identify the target PR:
   - If the user names a PR number, use it.
   - Otherwise detect the PR for the current branch: `gh pr view --json number,title,url`.
   - If no PR exists yet, ask whether to create one (`gh pr create`) or stop.
2. Show the final description to the user and confirm before publishing (updating a remote PR is a shared, externally visible action).
3. Update the remote PR body using the GitHub CLI:
   ```sh
   gh pr edit <number> --body-file <path-to-description.md>
   ```
   - Prefer `--body-file` over `--body` to preserve markdown, newlines, and special characters.
   - Write the drafted description to a temp file first, then pass it with `--body-file`.
4. Confirm success by reporting the PR URL returned by `gh`.

### Requirements & fallbacks
- Requires the GitHub CLI (`gh`) installed and authenticated (`gh auth status`). If not authenticated, tell the user to run `gh auth login` and stop.
- If `gh` is unavailable, fall back to any available VS Code GitHub Pull Request tools, or output the description for the user to paste manually.
- Never force-push, edit unrelated PR fields, or change the PR title unless the user asks.
