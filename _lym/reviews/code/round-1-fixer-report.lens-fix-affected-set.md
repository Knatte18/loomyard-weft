```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the F1, F2 and F4 fixes cover every member of the sets their premises name.

The review's findings are all fixed, and I found no gaps.

- **F1, `RaddleFileDeps` seam sites:** `AlreadyFiled` appears at every site that sets or lists `StoreDone`:
  - The struct and its call site in `loomshed/raddlefile.go`.
  - The real wiring in `loomcli/raddlewiring.go`.
  - The `requireSeam` list in `shedrecipe/entries_raddle.go`, with a nil-seam row in `entries_raddle_test.go`.
  - The `shedrecipe/fixture_test.go` fixture.
  - `envkit.go`.
  - The `loomshed/raddlefile_test.go` fixture.
  - The `battencli/wire_seams_test.go` exemption map.
- **F1, `RaddleEntry` bodies:** the only non-test constructors are `loomshed/raddlefile.go:372` and `:389`, and both now build bodies from the shared prefix helpers. The test helpers `raddlePart` and `raddleProject` use the same prefixes. Every `RaddleEntry` in `boardengine/raddlefile_test.go` comes from those two helpers.
- **F4, `ProposedSession` seeds:** outside `raddleverbs.go` and its test, `ProposedSession` appears only in `loomshed/raddlestate.go` and `raddlestate_test.go:31`. That test seeds `Session: 3, ProposedSession: 2, NoticeSession: 2` and never calls the approve verb, so it is unaffected. The `replace_all` edit covered every approve row in `raddleverbs_test.go`.
- **F2, docs:** the overview rows are in place at `docs/overview.md:142-143`, and no stale `raddle` note pointer remains. `run.json` and `landed.json` match the constants in `shedrun/usage.go:20-21`. The mentions of `raddle-approve` in `raddle-proposal-spec.md` and at `overview.md:535-536` describe the verbs without the old refusal condition, so nothing is stale.

Tool calls:
1. grep across the repo for `StoreDone`, `AlreadyFiled`, `ProposedSession`, `RaddleEntry{` and the overview rows. My context lacked a full-repo census of the seam and seed sets.
2. grep for the `run.json` and `landed.json` constants and for docs naming `raddle-approve` or the waiting notice. My context lacked a check of the docs row's file names and of other docs describing the approve refusal.
