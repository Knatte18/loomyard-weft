# Card 8 — cleanup-cancelled-record

**Edit:**
- `internal/battencli#cleanupSlug`
- `internal/battencli#judgeStatus`
- `internal/battencli#cleanupDeps`
- `internal/battencli#Cleanup`
- `internal/battencli#TestCleanup`

**Uses:**
- `internal/battencli#removeRunRecords`
- `plan:internal/runfsm#ParseRunRecord`

**Intent:**
`lym batten cleanup` repairs only a cancelled record a crash left (D3).
`judgeStatus` takes a record whose shed state is `cancelled` and whose cancel intent has a finish as a candidate beside a `done` one, read with `runfsm.ParseRunRecord`.
`cleanupSlug` removes such a record, through `removeRunRecords` as for a done run, only when the slug's pair is gone: a new `cleanupDeps.PairPresent` seam, set in `Cleanup` to the same pair probe the cancel uses.
While the pair stands it removes nothing and records a refusal in the report naming `lym fabric remove <slug>` as the way forward, which removes the pair and, through card 9, the record with it.
This lets cleanup remove one more kind of record, bounded to a finished cancel whose pair is gone; an unfinished cancel, a kept cancel whose pair stands, and every other state stay untouched.
`TestCleanup` (tier0) rows: a finished cancel with its pair gone is removed in apply and listed in a dry run, one whose pair stands is refused naming `lym fabric remove <slug>`, and a `cancelling` record is left.

**ImpactSummary:** cleanup also removes a finished cancel's record once its pair is gone, and refuses it while the pair stands.

**Commit:** 8: lym batten cleanup removes a finished cancel's record once its pair is gone
