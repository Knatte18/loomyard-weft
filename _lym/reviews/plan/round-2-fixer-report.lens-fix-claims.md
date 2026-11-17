```yaml
lens: fix-claims
outcome: reported
```

Task: check that each claim the fix added or rewrote holds against the code at HEAD.

**MEDIUM** — card 23, bring-up paragraph: "The report key `owner_pauses`, `orchcli.startReport.OwnerPauses` and `beamcli`'s `startEntry.OwnerPauses` keep their names and shape".
- **Problem:** `orchcli` has no `startReport` type. The field is `BeamUpReport.OwnerPauses`, so the card names a type the code does not have, and the implementer has to guess which struct is meant.
- **Evidence:** `internal/orchcli/start.go:67` declares `type BeamUpReport struct`, and its field is `OwnerPauses []map[string]string \`json:"owner_pauses"\`` at `start.go:89`. Nothing in `start.go` declares `startReport`. `beamcli`'s `startEntry` does exist, at `internal/beamcli/cli.go:427` with `OwnerPauses` at `:438`. The wrong name comes from the review's fix key and the F2 ruling.

These other claims hold:
- **`isBeam` source:** `boardengine.ReadOwners(...).Entries[slug].Beam` matches `Owners.Entries map[string]OwnerEntry` and `OwnerEntry.Beam` (`owners.go:29,50`).
- **Nil seam warning:** a nil seam is skipped and warned today (`start.go:426`).
- **Warning wording:** the reworded warnings keep the shape of the current texts (`start.go:427,431`).
- **`EntryOutcome`:** it has no beam flag (`originreconcile.go:268`).
- **`PairPresent`:** it is a path check, `markerExists` = `os.Stat` (`cancelmarker.go:123`).
- **Hand-over removal:** the hand-over removes a pair only when it is a registered linked worktree, since `taskWorktreeGone` lists worktrees (`teardown.go:811`) and `HandOver` skips removal when `pair == nil`.
- **Card 15 ordering:** the claim that the refusal comes before `req.Clear` is consistent with `Resume`'s present order (`resume.go:122-137`).

Tool calls:
1. grep of `boardengine` `Owners`/`OwnerEntry`, the report types, `markerExists` and `TaskWorktreeGone`: my context lacked their definitions.
2. `start.go:80-90` and the `taskWorktreeGone` lookup: needed the report struct's name and the predicate's body.
3. The struct list in `orchcli/start.go` and the body of `taskWorktreeGone`: needed to confirm whether `startReport` exists and what "gone" reads.
