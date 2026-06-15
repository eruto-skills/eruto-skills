# RUN — Autopilot Loop Body (dogfood run)

You are running inside an autonomous loop on the eruto-skills/autopilot installer task.

## Step 0 — Deterministic stack

1. Read `.claude/autopilot/PLAN.md` in full.
2. Read `.claude/autopilot/LESSONS.md` in full.
3. Read `.claude/autopilot/BUDGET.md`. If any cap is exceeded → `<status>BLOCKED</status>` and stop.
4. Run `git status --porcelain`. Uncommitted foreign changes → `<status>NEEDS_HUMAN</status>` and stop.

## Step 1 — Pick one task

Lowest-priority `passes: false` task from PLAN.md. **Exactly one.**

If none → `<status>DONE</status>` and stop.
If PLAN.md "Open questions" non-empty → `<status>NEEDS_HUMAN</status>` and stop.

## Step 2 — Premise check

Read the task's acceptance criterion. If any are true, emit `<status>NEEDS_HUMAN</status>`:

- Acceptance is not user-observable
- Required dependency missing (e.g., WSL unavailable for T02)
- Description and acceptance disagree
- Task would require expanding PLAN.md scope

## Step 3 — Implement

- Subagents OK for design exploration or code review.
- Only one build/test at a time.
- Smallest change that could plausibly make the acceptance test pass.
- Never edit files outside the task's scope (here: `autopilot/scripts/` for T01/T02, `autopilot/README.md` for T03 only).

## Step 4 — Run acceptance

Execute the exact commands in the task's `acceptance` block, in order. Record pass/fail per step in the task's `notes`:

```
- 2026-MM-DD HH:MM step <n> PASS  <command>
- 2026-MM-DD HH:MM step <n> FAIL  <command>: <one-line excerpt>
```

If any step in the acceptance fails, the WHOLE task fails this iteration.

## Step 5 — Branch on outcome

### 5a — Acceptance passed

1. `git add <specific files>` (NOT `-A`)
2. `git commit -m "autopilot: T<id> — <task description>"`
3. PLAN.md → set this task's `passes: true`
4. Append a durable rule to LESSONS.md if you learned something (1–3 lines, dated)
5. Increment BUDGET.md `iterations_used`
6. Emit `<status>CONTINUE</status>` and stop

### 5b — Acceptance failed

1. Append the failure line to the task's `notes`
2. Count consecutive failures in `notes` for this task:
   - 1st: revert untracked changes (`git restore .` on tracked files we touched; delete created files we didn't commit). Emit `<status>CONTINUE</status>`
   - 2nd: same, plus write a one-line hypothesis about why the prior attempt failed
   - 3rd consecutive: emit `<status>BLOCKED</status>` and stop. Do not try again.

## Step 6 — Status output is the LAST line

Exactly one of `<status>DONE</status>` / `<status>CONTINUE</status>` / `<status>NEEDS_HUMAN</status>` / `<status>BLOCKED</status>` on the final line.

## Anti-patterns (immediate `BLOCKED`)

- Editing PLAN.md to lower the bar
- Marking `passes: true` without running acceptance
- `--no-verify`
- `git add -A` / `git add .`
- Silent pivot to a different task
- Mocks/stubs to satisfy a user-observable test
- "Just one more try" after 3 failures
