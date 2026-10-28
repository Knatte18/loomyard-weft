```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced what each decision's state meets in code that already runs.

**Paths traced, per decision**

D1, Finalize merges in place:
- Finalize's resolver is built for every recipe, conflict session included, so loom's Finalize already has one to call (`internal/landingshed/finalize.go:154-174`). Survives.
- `premerge.clear` clears a parked merge-in before each merge, the retry pass included (`internal/landingshed/doc.go:70`). A resolver merge parked by an interrupted conflict session is cleared on the next pass. Survives.
- The retry after a merge-in-required error goes through `mergeAndGate` and so takes the new merge path (`internal/landingshed/finalize.go:319-336`). Survives.
- The merge record's `Overlap` is still written, and the fixer reads it. Survives.

D2, flags, `taskMerger` and overlap note removed:
- Darn's `ReviewInput` override goes away and the shared Parent-Align fixer-report seam answers not found on darn. Survives: the record states it.
- A stale overlap note left in an upgraded darn run's scratch dir now has no reader. Survives: the record accepts the loss.
- `envkit.go:153` sets `ConflictReroutes: true`. That is a call site, which is not this lens's concern.

D3, escalate prefix and no hand-back:
- `shedverbs` `Spec.handsBack` writes `hand_back` into the status and the loop envelope from `arbiterstop.HandsBack` (`internal/shedverbs/spec.go:228`, `status.go:187`, `loop.go:335`). Reads right: the escalate is not marked.
- `arbiterstop` sets `Stop.LandingHalt` from `landingshed.HandsBack` (`internal/arbiterstop/arbiterstop.go:164`). Reads right: the escalate is eligible as a blocked stop.
- The driver's `stopFromStatus` `HandBack` mark (`internal/sheddriver/driverules.go:115`). Reads right.
- Batten's `HandBack` seam is `arbiterstop.HandsBack` (`internal/battencli/wire.go:1066-1068`), and only the binary re-step reads it (`internal/battenshed/innerrun_binaryrestep.go:63`). D3's skip is placed right. Survives.
- Halted reasons from an older build keep `LandingStopPrefix` and stay hand-backs. Survives.

D4, `JudgesFirst(producer, reason)`:
- Envelope-built stop: `Reason` and `Error` are both decoded from the envelope (`internal/sheddriver/driverules.go:86,92`). Survives, assuming the envelope's reason holds the halt text.
- Status-built stop: `stopFromStatus` leaves `Reason` empty for every non-paused state and carries the halt text only in `Error` (`internal/sheddriver/driverules.go:110-121`). It is used at the moved-run recheck (`internal/sheddriver/drive.go:776,781`), the start check and the failure hand-back. **Breaks**: see the finding below.
- The arbiter's resume goto onto Finalize or Parent-Align: the loom goto guard admits both rows, except Finalize while the parent checkout is dirty, which it words with its own refusal (`internal/loomcli/finalizeguard.go:19-34`, `settledguard.go:19-24`). The settled-segment arm covers only review and fix rows. Survives.
- The escalate gives its attempt back (`s.rec.Spent--`, `internal/landingshed/fixerrun.go:256-262`). Only the arbiter spawn cap bounds an escalate-and-resume loop, as D4 states. Survives.
- The build hand-over row reads settle records per stop key, and a re-escalation gets a new history length and so a new key. Survives as D4 states.

D5, docs: no runtime path. Out of this lens.

**Findings**

1. MEDIUM, class `design`, D4 (`decision-record.md`, D4's first paragraph: "the reason being the stop's reason, which the driver's stop already carries").
   Summary: a status-built stop carries the halt text in `Stop.Error` and an empty `Stop.Reason`, so `JudgesFirst` fed `Stop.Reason` misses the landing escalate on every status-built stop. Such a stop falls through to the tackler, which contradicts D4's "with no tackler call" and the acceptance criterion "The driver sends such a stop to the arbiter with no tackler call".
   The driver's other prefix rows already read both fields for this reason, for example `budgetExhausted` with `stop.Reason || stop.Error`. D4 should say the seam is handed the reason, or for a stop built from a status its error, as those rows do, and an acceptance criterion should cover a status-built escalate stop.
   Evidence: `internal/sheddriver/driverules.go:109-125`, `driverules.go:248-254`, `drive.go:713-714`, `drive.go:776,781`.

**Tool calls**
1. Grep for `HandBack(` in `shedverbs`, the envelope's `Reason` and `Error` decode, and the `LandingHalt` writers: to find the readers of the hand-back mark one hop past the manifest.
2. `internal/shedverbs/spec.go` `handsBack` and the envelope decode in `driverules.go`: to confirm which stop fields carry the reason.
3. `drive.go` `handBackFailure` and `checkBlockedStart`: to see which paths build a stop from status.
4. Grep for goto guards: to check whether the arbiter's resume goto onto a landing row could be refused.
5. `finalizeguard.go` and `settledguard.go`: the guard bodies, to judge whether that goto survives.
