```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency lens: checking that the fixer's changes agree with the cards, rows and cross-references around them.

**MEDIUM: card 2's PATTERN background edit leaves its PATTERN.md entry stating the old rule**
- Changed text: card 2's new sentence rewriting `pattern/PATTERN-run-process-owner.md`'s End-only lines, including its opening "ended only through `runproc.End`".
- Issue: the entry's one-liner in `PATTERN.md` still says the opposite of the rewritten background.
- Evidence: `PATTERN.md:55` reads "`PATTERN-run-process-owner` — A run's processes: registered in `internal/runproc`, ended only via `runproc.End`."
  Card 2 does not list `PATTERN.md` among its targets.
  The overview's "PATTERN changes" section names only `PATTERN-owner-machine`.

**MEDIUM: card 25 changes `cancelFake` without listing it as a target**
- Changed text: card 25's tests paragraph, "`cancelFake.deps`' `Owner` returns the fake's subject", and the `cancelDeps.Owner` re-sign to `(machine, owner, subject string, err error)`.
- Issue: for the fake to return a subject, the `cancelFake` struct needs a subject field.
  Card 25's Edit list has `internal/battencli#cancelFake.deps` but not `internal/battencli#cancelFake`.
- Evidence: card 20 lists both `cancelFake` and `cancelFake.deps`, so the twin was missed in card 25.
  Today `cancel_test.go:118` returns `f.machine, f.owner`, and the struct has no subject field.

**LOW: card 20's opening summary no longer matches the new held-hold wording**
- Changed text: card 20's new bullet on `refuseBeamCancel` once the hold reads held (cancel, then board move, never `lyx batten run`), and its two new `cancelDeps` fields.
- Issue 1: the card's opening paragraph still says the cancel "refuses while `BeamLandRefusal` refuses ... naming the members to land or cancel first".
  That clause still holds only while the hold is not held.
- Issue 2: the first Intent bullet's field list says "`hubCancelDeps` fills them, and a nil `BeamProject` reads as no beam".
  It does not cover the new `LandingHeld` and `LandBlockers` fields, and nothing says what a nil one does.
- Evidence: card 20, the Intent's first paragraph and its `cancelDeps` bullet.

Tool calls:
1. Grepped the plan cards for the changed symbols (`ownedElsewhereRefusal`, `cancelFake`, `DobbyLive`, `landHaltReason`, the no-local-beam-pair refusal, `resumeOwnerClaim`), to find unchanged copies elsewhere that my context lacked.
2. Grepped `PATTERN.md`, card 0 and card 10 for `PATTERN-run-process-owner` and `EndRole`, because `PATTERN.md`'s entry line was not in my context.
