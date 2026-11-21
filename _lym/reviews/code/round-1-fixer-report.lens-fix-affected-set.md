```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the searches behind F1, F5, F2, F3 and F7, and the fix covers every member I found.

No findings.

- **F1:** the only Go `"No test seam"` literal is now `gateslot.NoTestSeamTitle`, declared in `internal/gateslot/redfirst.go`. The other hits are contract and stencil markdown and prose comments, not code constants.
- **F5:** no kept-findings comment still describes clearing. The sets I searched were `shedrun`, `shedadapters` (including `burlerfix.go`, `multillm.go` and `doc.go`), `shedrecipe`, `loomshed`, `seatengine` and `burlerengine`. Every `gateFindingsPath` / `KeepGateFindings` comment now says "kept".
- **F2:** `redfirstgate.go` is the only non-test closure that returns `Pending: true` with a `Send`, so no sibling pending closure needed the `Sent` hook. `shuttleengine/doc.go` mentions neither `Send` nor `SendFailedWayForward`, so it has no stale text about the new field.
- **F3:** `FindingsPath` is read only in `latestDarnOutcome` and `renderPriorWork`, and the age check sits in the one producer, so no other consumer needs it.
- **F7:** `preflightTableError` has a single call site, now through `c.env.DarnTable`.

Calls made:
1. Grepped `No test seam` across Go and markdown, and the stale clearing phrases across the repo; the stale-phrase pattern was too broad and mostly matched unrelated "removes it" lines.
2. Grepped `Pending: true` literals, `preflightTableError` and `FindingsPath` uses. I needed this because my context held only the files the review named, not every closure and caller.
3. Grepped `gate-findings` and `GateFindingsFile` consumers outside `shuttleengine`, to find sibling users of the findings file.
4. Grepped `shedadapters`, `shedengine` and `shedrecipe` for `KeepGateFindings` and clearing code or comments, to confirm no remover or stale comment remains.
5. Grepped `shuttleengine/doc.go` for pending-send wording, to see whether it needed the new `Sent` field.

I used a sixth search inside call 1 (the `shuttleengine.GateResult{...Send:` pattern, which returned nothing).
