```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the set-premise searches for the findings that name a set of callers, sites or keys.

**Findings**

- **LOW** — `internal/shedadapters/parentseat.go` `due()` (`LatestTagged(a.seat.Tag)`), a F14 site.
  - Summary: asks recorded before the tag existed carry an empty `Tag`, so no seat's own-kind lookup finds them.
  - Evidence: a Discussion-Review generation re-entered on a run with an old untagged ask finds none and asks once more, the case F14 meant to stop.
  - Bounded: it affects only in-flight runs, once.
  - No code change is needed unless the one repeat matters.

**Searches by finding**

- **F1, stale references:** a grep for `AskEachRound`, `roundParentAsk`, `settleRoundParentAsk`, `heldParentAsk` and the old "each round / round before's review" phrasing finds nothing stale. The one hit is `contracts/orchrole/orch-role.md:245`, "for every round", which is still accurate.
- **F1, every mention of the code-review keys and tag:** these files all carry the updated Bouncer-timing text.
  - `internal/loomcli/doc.go`
  - `internal/shedadapters/doc.go`
  - `internal/loomengine/template.yaml`
  - `internal/loomengine/config.go`
  - `contracts/orchrole/orch-role.md`
  - `docs/overview.md`
- **F1, Bouncer construction:** the only `BouncerConfig{` site is `internal/shedrecipe/entries_bouncer.go:282`, and it now sets `Parent`. Discussion-Review Bouncers get a seat with `AskBeforeJudge` false, so they ask nothing.
- **F1, `RoundAsk` consumer:** `roundcontext.go:78` finds the previous round's report by seed and round, which still holds for the Bouncer's asks. `roundProfile` passes an empty anchor to `BuildRoundContext`, so it is unaffected either way.
- **F2, `ParentAgreementGateEntry`:** its one non-test caller, `verify.go:848`, passes the new `limit` argument.
- **F2, ask record writers:** the writers of `ParentAsk` are the judge (`judgeparent.go:157`, with Tag and Digest) and the seat asker (`parentseat.go:200`, with Tag). No other writer exists.
- **F14, readers of the one ask record:** `Latest()` is used only in the answer and expire paths of `burlerengine` (answering and expiring the run's latest ask, which is correct). `reportengine`, `landingshed` and `loomcli` read the record only through the answer and expire verbs. No other reader is affected by the new fields.
- **F3:** the ruling's keys all stay `CategoryNone`, and the PATTERN text now carries the exception. Nothing in the code needed to move.
- **F12, F13:** the refusal text and the `ParentAsk` docs have no other copies.
- **F4:** held by the parent, with no change. Left unchecked.

**Tool calls**
1. Bash grep for stale F1 references, ask-record users and key mentions: the set searches for F1, F2, F3 and F14.
2. Bash grep for other readers of the ask record, test files building asks, and `BouncerConfig{` sites: the F14 and F1 consumer sets.
3. Read of `internal/shedadapters/roundcontext.go:55-115`: it was not in my context, and it is a `RoundAsk` consumer.
