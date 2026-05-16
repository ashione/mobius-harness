---
name: local-repo-development
description: Follow a local repository development workflow with git worktree setup, pre-commit code review, and sensitive information scanning.
---

# Local Repo Development

## Intent

Follow a conservative local repository development workflow that isolates feature work in git worktrees and gates commits with review and secret scanning.

## Inputs

- `feature_name`: Short feature, fix, or task name used to identify the worktree and branch.
- `repo_path`: Local repository path, usually the current working directory.
- `commit_intent`: Optional summary of the changes that should be committed.
- `mr_url`: Optional merge request or pull request URL created for the work.

## Instructions

1. Confirm the current directory is inside a git repository with `git rev-parse --show-toplevel`. If it is not, stop and ask for the repository path.
2. Before editing, determine whether the current checkout is already a linked worktree for this task:
   - Inspect `git worktree list --porcelain`.
   - Inspect `git rev-parse --git-dir` and treat paths under `.git/worktrees/` as linked worktrees.
   - If already in a suitable linked worktree, continue there.
3. If not in a suitable linked worktree, create one before making changes:
   - Derive a kebab-case slug from `feature_name`.
   - Prefer base ref `origin/main`, then `origin/master`, then local `main`, local `master`, then `HEAD`.
   - Create a feature branch such as `feature/<slug>` or `fix/<slug>` based on task intent.
   - Place the worktree in a sibling directory such as `../<repo-name>-<slug>` unless the repo has a documented worktree location.
   - Use `git worktree add -b <branch> <path> <base-ref>`. If the branch or path already exists, choose a non-destructive unique name.
4. Never discard existing user changes while preparing the worktree. If the current repo or selected worktree has unrelated dirty files, keep them and work around them; ask only if they block the task.
5. Implement the requested change in the selected worktree using the repository's existing build, style, and test conventions.
6. Before committing, run a code review pass over the local diff:
   - Review `git diff HEAD` or the staged diff for bugs, regressions, missing tests, unsafe behavior, and unintended file churn.
   - Fix material findings before continuing.
7. Before committing, scan for sensitive information in the local repository changes:
   - Prefer an installed scanner such as `gitleaks detect --source . --no-git --redact` or `detect-secrets scan`.
   - If no scanner is available, do a focused fallback scan of changed files for common secret patterns such as API keys, private keys, tokens, passwords, `.env` values, and credentials.
   - Do not print secret values. Report only file paths, key names, and redacted evidence.
8. Stage only intentional files, then commit with a clear message after the code review and sensitive information scan pass. If either gate cannot be run, state that explicitly before committing.
9. After submitting a merge request or pull request, asynchronously track CI/CD until it reaches a terminal state:
   - Record the MR or PR URL and the branch name.
   - Check pipeline status with the repository's normal tool, such as GitHub checks, GitLab pipelines, or the project's CI dashboard.
   - Do not block unrelated work while waiting, but revisit the pipeline until it passes, fails, or is canceled.
   - If CI/CD fails, inspect the failing job logs, summarize the failure, and fix issues that are in scope for the change.
   - Report the final CI/CD state clearly, including any checks that could not be observed.

## Examples

Input: Add a dashboard filter feature in `/repo/app`.

Output: Confirmed current checkout was not a linked worktree, created `../app-dashboard-filter` from `origin/main` on branch `feature/dashboard-filter`, implemented the change, reviewed the diff, ran a secret scan, committed only the intended files, opened an MR, and asynchronously monitored CI/CD until the pipeline passed.
