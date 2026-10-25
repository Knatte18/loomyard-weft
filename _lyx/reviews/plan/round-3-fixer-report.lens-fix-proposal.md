```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged the proposals behind F1, F2, F4, F5 and F6 as the fixer carried them out.

**LOW: F2, step 7 of `11-drift-verify-producer.md`, in "A resume that halts at step 3, 4, 5 or 6 leaves `gate_halt_at` set too".**
Step 3 cannot halt, so the list names a halt that does not exist.
Step 3 only marks the named tests `ruled` and clears `halt_named` and `halt_named_at`.
Steps 1 to 6 as written halt only at 4, 5 and 6, and step 1 ends in Done, never a halt.
The text came over from the reviewer's proposal.
It should read "steps 4, 5 or 6".

**LOW: F2, step 7, in "A failure is Stuck … and rewrites `gate_halt_at` in one `UpdateScope` call".**
A failing resume still rewrites the field, so the window the finding is about stays open, only narrower.
Concrete case:
1. Entry E1 halts with a gate Stuck at second S. The history's last entry is that Stuck, and `gate_halt_at` is S.
2. Resume E2 runs the gate, which fails. E2 writes `gate_halt_at` as S′ &gt; S and is then interrupted before the shed persists its Stuck. The persist runs after `Call` returns (`internal/shedengine/run.go`: history is appended only from the outcome, after `def.Producer.Call`).
3. E3 reads the last history Stuck at S, which is earlier than S′. E3 is no resume, clears the field, skips the gate and compares code that was never gated.

The same thing happens on a failure inside a resume.
If the failure kept an existing `gate_halt_at` and wrote one only when none was set, E3 would still be the resume, because S is at or after S.
It would run the gate again.
The new Stuck at S″ is later than the kept time, so the next resume still reads correctly.

F1, F4, F5 and F6: I built no case where the proposed change is wrong.
- F1: both sides are now at second precision, so ties resolve to the resume as intended.
- F4: no card reads `gate_halt`. The opening texts stay as constants for step 10's reasons.
- F5: the equal-head row agrees with step 7's "not due (`AttemptHead` absent or equal to HEAD)".
- F6: a later step-4 halt writes a fresh `halt_named_at`, so clearing it in step 3 loses nothing.

Tool calls:
- I read `internal/shedengine/run.go:300-360` to confirm that an empty-outcome error records no history and that a Stuck is persisted only after `Call` returns. My context showed only the `nowRFC3339` grep.
