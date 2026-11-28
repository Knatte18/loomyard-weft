```yaml
lens: fix-code-correctness
outcome: reported
```

Task: correctness check of the code changed by the fix (lens `fix-code-correctness`).

no findings

What I traced:
- **`strandsElsewhereLocked`**:
  - With no absent pane, it returns two nil maps.
  - When `panePlacements` fails, it returns nil `elsewhere` and `unread` holding exactly the absent ids.
  - On success, the named `elsewhere` starts nil and is made on first use, and `unread` is nil.
  - Both callers, in `lifecycle.go` and `reconcile.go`, take the new signature. `reconcile` plans with the same input as before.
- **`downCause`**:
  - After a failed lookup, `%9` and `%10` read `pane-placement-unread`.
  - `%6` reads `pane-dead` and `%7` reads `start-shell`, since both are listed and so not in `unread`.
  - The empty pane id is never put in `unread`, so `unbound` reads `pane-unlisted`.
  - Reading a nil `unread` map is safe.
- **Tests**: the failed-lookup assertion would fail with the old code, because `gone` and `nowhere` would read `pane-unlisted`. The `unbound` row covers a gap in coverage and fixes no defect, so it has no failing-before state to show.
- **Help string and log constant**: the concatenation yields the same cause names plus the new one. Both log calls keep their level and arguments.

The parent reported the gate passing for the three packages before the first commit and for `orchroute` after the second. I did not rerun it.

Tool calls: none.
