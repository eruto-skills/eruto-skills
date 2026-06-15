# LESSONS

> Cross-iteration durable rules. Read at top of every iteration.

## 2026-05-13 (dogfood scaffolding observations — not produced by the loop yet)

- Rule: For artifacts at `.claude/autopilot/`, "the current project" is the cwd when `/autopilot` was invoked. For multi-skill workspaces like eruto-skills/, this is the workspace root.
- Why: SKILL.md didn't disambiguate during dogfood scaffolding.
- How to apply: Whenever the skill writes artifacts, it should explicitly print the destination path to the user during step 3.

## Entries

## 2026-05-13 — Iteration 1, T01 pre-flight

- Rule: Step 0.4 "git clean check" must be scoped to files the current task can touch, not the whole repo.
- Why: Multi-project workspaces almost always have pre-existing dirty state from unrelated work. Checking the whole tree generates false NEEDS_HUMAN on iteration 1.
- How to apply: Compute the union of paths implied by the active task's acceptance + description. Only fail if THAT subset is dirty by another author.

## 2026-05-13 — Iteration 1, T01 post-pass

- Rule: Skill must obtain explicit "commit authorization" during step 2 interview before the loop is allowed to git commit.
- Why: Global Claude Code policy says "only commit when explicitly requested". RUN.md unilaterally commits in step 5a. Conflicting policies = either the skill silently violates user expectations, or the loop stalls every iteration waiting for permission. Found during T01 of dogfood.
- How to apply: Add a 5th interview question: "Authorize the loop to git commit on each passing task? (y/N)". Persist answer to BUDGET.md `auto_commit: bool`. RUN.md step 5a reads it; if false, only updates PLAN.md and prints a "ready to commit: <task id>" hint.

- Rule: Step 3 implementation should use `Split-Path -Parent $PSScriptRoot` (not hardcoded paths) so the installer works regardless of where the user cloned the repo.
- Why: Earlier draft used $repoRoot from an env var; not portable. Switched to PSScriptRoot — works from any location.
- How to apply: For PowerShell scripts in skills, derive paths from $PSScriptRoot.

## 2026-05-13 — Iteration 2, T02 post-pass

- Rule: When acceptance commands shell into WSL from Windows, use `wsl.exe -- bash << 'EOF' ... EOF` (heredoc), NOT `wsl.exe -- bash -c '...'`.
- Why: The `-c` form's nested single-quoting around multi-line scripts with `set -e` + `$(...)` substitution + `2>&1` mangles in unpredictable ways. Heredoc passes the script verbatim.
- How to apply: Whenever T-acceptance is a multi-line bash script invoked via wsl.exe, prefer heredoc.

- Rule: install.sh must handle macOS BSD `readlink` (no `-f`) — fallback chain: `greadlink` → GNU `readlink -f` → `perl -MCwd=abs_path`.
- Why: macOS doesn't ship `readlink -f`. WSL Ubuntu does, but a real macOS user would hit this.
- How to apply: Any portable bash script needing canonical path resolution should use this chain.

## 2026-05-13 — Iteration 3, T03 post-pass

- Rule: Acceptance criteria using `git diff` are only verifiable if the file is git-tracked. If untracked, fall back to manual inspection or use a different check (e.g., re-Read the file and diff against a known-good).
- Why: T03 step 3 wanted "only the Install section changed", but autopilot/README.md is untracked in this workspace, so git diff is empty. Spirit-pass was acceptable here, but a real autonomous loop would either fail it or skip silently — both bad.
- How to apply: Before writing acceptance criteria, check that prerequisites (tracked state, server running, env vars set) are met. RUN.md Step 2 premise check should add this.

<!-- new entries appended below by the loop -->
