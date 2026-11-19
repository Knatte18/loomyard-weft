Task: check the change set through the fix-consistency lens, for sibling text, docs and registrations left out of step.

**Result: no findings.**

- `judgeParentBlock` has no remaining reference in `.go`, `.md` or `.yaml` outside the excluded `_lym`, `.lym`, `.scratch`, `docs/research` and `.git` paths.
- The new `bouncer-fragment-parent-review` fragment is registered in three places: the embed variable, the registry entry and the `markers_test.go` row. These are the same three the other `bouncer-fragment-*` stencils use.
- The `doc.go` "Roles" passage says only that "Both prompts carry the parent directive". That stays true, so it needs no edit.
- The `doc.go` parent-seat passage already says the Bouncer asks before the judge, tells the judge to wait, settles the ask after the ruling and refuses `CONVERGED` over findings. The change to `settleUnlessRefused` makes that passage hold on the recovery paths too, so it still agrees.
- The assembled prompt text is unchanged. The block used to be one string starting `"\n\n## …"` and ending `"\n"`, and the code now builds the same bytes as `"\n\n" + block + "\n"` from the trimmed fragment.
- The replay at `bouncer.go:526` (`standsAsRecorded`) calls `settle` without settling the ask, but the round was escalated or decided on an earlier call that already went through `settleUnlessRefused`. Nothing is left stale.

Tool calls: two `Bash` calls, one grep for `judgeParentBlock` and sibling stencil references, one read of `doc.go` for stale parent-ask descriptions. Context held neither the grep results nor `doc.go`.
