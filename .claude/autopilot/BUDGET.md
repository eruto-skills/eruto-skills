# BUDGET

## Caps

```yaml
max_iterations: 5            # 3 tasks × ~1.5 iter each
max_cost_usd: 5              # tiny task, sub-$1 expected
max_wallclock_h: 1
max_consecutive_failures: 3
min_progress_iterations: 4   # if no task passes within 4 iterations, BLOCKED
```

## Counters

```yaml
iterations_used: 3
started_at: 2026-05-13T(iter1)
last_pass_at: 2026-05-13T(iter3)   # T03 passed
estimated_cost_usd: ~0.65
```

## Halt notifications

```yaml
notify:
  slack_webhook: ""
  email: ""
  desktop: true              # Windows BurntToast — implement when needed
```

## Notes

This is a dogfood test. The dominant goal is **observing autopilot's friction**, not minimizing cost. Caps are tight on purpose so we don't drift into "fix everything autopilot doesn't do" scope creep.
