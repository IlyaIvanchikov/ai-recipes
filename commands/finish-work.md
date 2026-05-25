# finish-work

Tears down a git worktree safely after the ticket is done — removes the
worktree directory, optionally deletes the local branch, prunes stale
metadata.

## When to use

After a feature branch is merged or its PR is closed and you want to free
up disk space and tab clutter without losing work to a careless `rm -rf`.

## How it works

1. Resolve the worktree to remove — from an argument (branch name or
   path), or by listing `git worktree list` and asking the user.
2. Safety check: run `git status --porcelain` inside the worktree.
   If there are uncommitted changes, warn and require confirmation.
3. Safety check: run `git log @{upstream}..HEAD` to look for unpushed
   commits. If found, warn and confirm before proceeding.
4. Capture the branch name (`git branch --show-current`) for cleanup.
5. Remove the worktree: `git worktree remove {path}`. If git refuses
   because of modified files and the user already confirmed, retry
   with `--force`.
6. Optionally delete the local branch: `git branch -d {branch-name}`
   (safe — fails if unmerged). If unmerged, offer `git branch -D` (force)
   with an explicit confirmation.
7. Prune stale worktree metadata: `git worktree prune`.
8. Report what was removed and show the remaining worktrees.

## What you'll need

- Git with worktree support
- A worktree-per-feature workflow (use this as the cleanup half of
  [start-work](start-work.md))

## Why this pattern works

The two safety checks (uncommitted changes + unpushed commits) catch the
classic "I thought this was merged but I had local fixes still" disaster.
Splitting the destructive step into two confirmation gates — first
`remove`, then `--force` only after a yes — means a confused user is
never one keypress away from losing work. Pruning at the end keeps
`git worktree list` honest; stale entries accumulate over time and
start lying about what's where.

## Variations

- **No interactive prompts (CI/scripting)?** Skip the warnings; just
  check exit codes from `git worktree remove` and fail loudly.
- **Always force-delete?** Don't. The branch-deletion safety net is what
  stops you from killing unmerged work.
- **Worktrees in non-standard locations?** Pass an absolute path instead
  of letting the script assume `./worktrees/{name}/`.
