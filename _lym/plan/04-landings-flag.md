# Card 4 — landings-flag

**Edit:**
- `internal/dobbyengine#State`
- `internal/dobbyengine#daemon.countLandings`
- `internal/dobbyengine#TestCountLandingsAndTidy`

**Extends:**
- `internal/dobbyengine/doc.go`

**Uses:**
- `internal/dobbyengine#newTidyHarness`
- `internal/dobbyengine#tidyHarness.step`
- `internal/dobbyengine#tidyHarness.writeReport`

**Intent:**
Decision 8's one-time emptying of a record written before the landing count read machines.

`State` gains `LandingsMachineOnly bool` with the tag `json:"landings_machine_only,omitempty"`, documented as: the record's `LandedSinceTidy` holds only this machine's landings.
It is tolerant: a record without it decodes as false (PATTERN-tolerant-records).

`countLandings`, inside its `UpdateState`:
- a record read for the first time baselines as now and also sets `LandingsMachineOnly`, since its list starts empty;
- while `Tidy` is in flight it changes nothing, as now, so a tidy in flight when the binary arrives settles its own slugs first (Decision 4);
- otherwise, when `LandingsMachineOnly` is false, it sets `LandedSinceTidy` to nil and the flag to true, then counts the fresh landings as now, so the emptying lands before `tidyStep` decides whether a tidy is due;
- with the flag set it counts as now and never empties again.
Only `settleTidy` and this one emptying clear the list.

`TestCountLandingsAndTidy` gains rows:
- a baselined record without the flag whose `LandedSinceTidy` holds `other`, with `board_tidy_every` 1 and no fresh landing: one step leaves the list empty, the flag set and no agent started;
- after that, a fresh landing `mine` makes the next step's list exactly `[mine]`, and a further step with no new landing leaves it as it is;
- a record without the flag and with `Tidy` in flight, its `Deadline` an hour past the harness clock so the step does not time it out, keeps its list and stays unflagged; once the test writes the report and the next step settles the tidy, the step after it empties nothing new and sets the flag;
- a record read for the first time comes out baselined and flagged.

`doc.go`'s board-tidy section gains the one-time emptying and its bound: it also drops this machine's own landings counted before the deploy and not yet tidied.

**ImpactSummary:** One tolerant field on the state record and one branch in the landing count, read only by the tidy step.

**Commit:** 4: Empty a pre-machine landing list once before the first count
