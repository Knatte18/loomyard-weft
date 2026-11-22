```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: no finding rises above LOW. Both fixes cover the set the review named, and one stale twin remains outside it.

**F2: a twin of the corrected claim remains (LOW).**
- Changed text: `_lym/landing/summary.md`, where "gets only the marker" was replaced by the seeding explanation.
- The same claim stands in `internal/hubreconcile/reinterpret.go:45`: "An absent base file holds no value to carry across, so only its marker is written."
- The review says the walk seeds the absent orch.yaml first, so the reinterpretation reads a seeded file. That file is flagged when it sets no `remote_control`.
- That comment may hold as a statement about the function in isolation. A reader of the walk would still notice it contradicts the corrected summary. The review did not name it.
- The other hits for "absent base file" or "no orch.yaml" are `internal/configengine`, `internal/lenses`, `internal/battencli` and `docs/shared-libs/configengine.md`, which are unrelated to reinterpretations.

**F3: covered.**
- The only places the orch reinterpretation ID is declared or used are `internal/orchengine/reinterpret.go:12`, the registration at `internal/configreg/configreg.go:161`, and the new test.
- The new test asserts the literal ID, the count of one and a non-nil `Rewrite`, so no other site needs the same assertion.

**Unchecked:** F1 is a decision-class finding the parent holds, so I did not search it.

Calls made:
1. `grep -rnE "only the marker|no orch\.yaml|absent base file"` over `.md` and `.go` files, to find other statements of the F2 claim; my context lacked the rest of the repo.
2. `grep -rn "orch-remote-control-off-by-default\|RemoteControlOffReinterpretationID"`, to find every user of the orch reinterpretation ID for F3; my context lacked the rest of the repo.
