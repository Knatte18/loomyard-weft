```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that each set-shaped fix covers every member (fix-affected-set lens).

Unchecked list:
- F15: I did not re-check which test holds the assertion of each card 15 deletion. The parent's own span measurements cover the line counts. That leaves the estimate's set only partly rerun.

Findings:

1. **MEDIUM**: card 6's new bullet ("A call of `Snapshot` or `Extend` on a record value… becomes `fabriclayout.MutationsOf` over both entry lists") lists three sites and misses a fourth.
   - `internal/fabriccli/fabric.go:894` copies `rec := r.Mutations`, a record value, and line 899 calls `rec.Extend(warp.Mutations)`. Line 901 then passes `rec` to `errWithRecordFields`.
   - Once card 6 lands, `r.Mutations` is a `fabriclayout.Mutations`, which has no `Extend`, so the build fails there.
   - The bullet's general sentence covers the site in principle. But its explicit list reads as complete and omits this one.
   - Evidence: `grep -rn "\.Extend(" internal cmd` gives `internal/fabriccli/fabric.go:899: rec.Extend(warp.Mutations)`. The site reads `rec := r.Mutations` at `:894`.

2. **MEDIUM**: the same class of break sits outside the bullet and outside card 6's Edit set. This is a recorder method called on a record value, here `Append`.
   - `internal/orchcli/start_test.go:1186` calls `h.report.Mutations.Append(fabricengine.KindWorktreeRemoved, …)`.
   - `h.report` is a `fabricengine.OriginReport` (`start_test.go:157`), whose `Mutations` field becomes `fabriclayout.Mutations` under card 6. That type has no `Append`.
   - `internal/orchcli` is not among card 6's Edit targets, and card 4 moves `KindWorktreeRemoved`'s kind. The test stops compiling at card 6.
   - The fix's bullet frames the affected set as `Snapshot` or `Extend` only, so it understates the set. It should cover any recorder method on a record value, with this site turned into `MutationsOf` over a literal entry, and add `internal/orchcli#TestStart_ReconcilesTheHubFirst` or its package to card 6's Edit list.

F16's set is covered: card 15 gives a `Store` for all six fields. `FieldLocalRefs` and `FieldOriginRefs` take the walk order, and the other four are named at `15-snapshot-reader.md:86`.

Tool calls:
- grep for `Mutations.Extend` and `Snapshot` calls on record values, to rerun the card 6 site search, which my context lacked.
- grep for every `.Extend(` call, to widen that search past the variable names the review named.
- grep for `Append`, `AppendRef` and `Snapshot` on `.Mutations`, plus the type of `rec` in `fabriccli/fabric.go` and `destroy.go`, to tell recorder receivers from record-value receivers.
- sed of `fabriccli/fabric.go:888-904` and `orchcli/start_test.go:1178-1190`, to confirm the two extra sites' receiver types.
