```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward, which checks that every way forward the fix added can be reached from the state it names and makes progress.

- **MEDIUM.** Changed text: D4, the new owned-elsewhere paragraph ("records the run as owned elsewhere … runs no further pass of that run") and the narrowed pass rule ("for a run its state record names owned elsewhere, only the full pass at the lift").
  Summary: the fix doesn't say whether a run dobby has stopped passing still counts toward D5's `max_runs` ("fewer than `max_runs` runs it drives are in flight, halted ones included") and D13's in-flight backoff. If it counts, a few taken-over runs block every new start until cleanup removes them.
  Evidence: such a run's local batten status stays `running` and has no cancel marker, which is exactly D13's definition of in flight. The fix leaves it in the state record with no further pass. It can only end through cleanup after the other machine lands the run, and until then a dobby with `max_runs: 3` and three taken-over runs starts nothing.
- **LOW.** Changed text: D4, "each of which runs a full pass (D4's lift trigger, and adopt's own first pass); it drives the run again only when such a full pass exits 0."
  Summary: dobby can't see the exit code of adopt's first pass, because adopt runs it detached and dobby didn't start it. So that half of the way forward can't happen as worded, and only dobby's own lift pass can clear the mark.
  Evidence: `internal/adoptcli/adopt.go:1` says adopt "lifts the owner pause and resumes the run". The lift fires D4's lift trigger, and D12 defers that lift pass until the in-flight pass exits, so dobby's own full pass does eventually run. The sentence credits a pass dobby can't observe.
- **LOW.** Changed text: D4, "runs today's owned-elsewhere handling once: `tendercli.PauseOwnedElsewhere`, …".
  Summary: if that pause fails, nothing retries it. The run then has no owner pause, so no lift trigger can ever fire, and dobby runs no further pass. Only `lyx adopt` moves the run on.
  Evidence: today InnerRun's `ownedElsewhere` retries a failed pause at the next probe while the owner stands (`internal/battenshed/innerrun_owner.go:101-115` and its doc comment). Dobby's version runs once.
- **LOW.** Changed text: D2, "`lyx adopt` of a member task whose parent beam, the prime included, reads owned by another machine refuses, naming `lyx adopt` of that beam".
  Summary: when the parent is the prime, the way forward is `lyx adopt --prime`, not `lyx adopt &lt;slug&gt;`, so "naming `lyx adopt` of that beam" points at a form that doesn't exist for the prime.
  Evidence: `internal/adoptcli/adopt.go:107-108` reads "name a task slug, or pass --prime or --all", and the prime is `req.Prime`, not a slug.

Tool calls:
- grep of `internal/adoptcli/adopt.go` for pause lift and prime handling, plus `ResumeInWindow`'s location. My context had only lines 210-232 of adopt, not whether adopt lifts the owner pause or how the prime is named.
