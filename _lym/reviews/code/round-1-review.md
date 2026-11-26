---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/websterengine/recoverbatch.go#recoverSpawn
    summary: A recovery resumed after an asking wait never gets its question or the parent's answer. The asking strand is stopped, the fresh uncounted spawn's prompt carries neither, and the Master rung has no step that passes an answer on, so the resumed strand can ask again in an unbounded loop of uncounted waits.
    fix: "Carry the waiting record's need, and the parent's answer, into the resumed recovery. Add an answer input to `lym webster recover-batch` (for example `--answer <text>`), recorded on the waiting BatchState. In recoverSpawn, when waitsOnParent(prior) holds, render a block into the recovery prompt (internal/websterengine/recoverbatch.go:317, beside failureDigestBlock) that quotes prior.Digest.Need and the recorded answer. In contracts/stencils/webster/webster-template-master.md:140-141, tell Master that for a question need it re-runs `lym webster recover-batch <NN>` passing the parent's answer verbatim. Bound the loop: count a second asking wait of the same batch as a recovery, or refuse a resume whose answer is empty. Add a TestRecoverBatch row asserting the resumed spawn's prompt holds the need and the answer, and a row asserting that a repeated asking wait is bounded."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/websterengine/recoverbatch.go:872
    summary: Every dead/asking turn end that carries any message becomes waiting, not only one that asks a question. A Claude Stop event almost always carries last_assistant_message, so a recovery that ends on a summary such as all cards committed, with no report, now pesters the parent instead of taking the commit-and-count retry rule.
    fix: "Route to waiting only a turn end whose message asks the parent something. Either have the recovery prompt tell the strand to report `status: NEEDS_PARENT` with its question as `need`, and keep a bare turn end dead/asking with the message quoted in the dead digest's halt text; or keep the awaitTerminal branch but record the message on the dead digest (for example a `last_message` field that recover-batch's refusal and the halt note print) rather than turning every message into waiting. Add a TestRecoverBatch_SecondCall row for a turn end on a non-question message that pins the chosen outcome."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/planglyph/planglyph_test.go#TestValidateFormat_MissingMemberFindings
    summary: Fix commit 4cede865 lists the sibling members of a missing member of a Go external-test `<dir>_test` unit from its package directory, but no test row covers that case, which failed as quarry unavailable before the fix.
    fix: "Add a row to TestValidateFormat_MissingMemberFindings whose fixture holds `sub/a_test.go` in `package sub_test` with two funcs, and a card using `sub_test#Missing`. Assert the exact glyph-not-found detail, members beside it listing the two external-test funcs and none of package sub's own."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/websterengine/recoverbatch.go:873
    summary: A LastTurnMessage error returns before StopStrand, which breaks the documented rule that every terminal digest removes the recovery strand. The live strand and the in-flight record stay behind.
    fix: "In awaitTerminal, on a LastTurnMessage error keep the dead/asking digest and append the error to warnings instead of returning, so the StopStrand call below still runs. Alternatively, move the asking-to-waiting conversion after the StopStrand call. Either way, keep the doc at recoverbatch.go:805 true."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatefixer/strand_test.go#TestLastTurnMessage
    summary: The mutation survivor at internal/gatefixer/strand.go:43 shows that no row checks that the bytes from the offset on are the ones parsed, and no row has an offset past the end of the file.
    fix: "Add a row whose content is `earlier-stop\\nlater-stop\\n`, with offset len(`earlier-stop\\n`) and a fake that returns a stop with message `later` only when data holds `later-stop`. Want `later`. Add a row with an offset beyond len(content) whose fake returns a stop only for empty data. Want that stop's message."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/planglyph/planglyph_test.go#TestValidateFormat_MissingMemberFindings
    summary: No test covers several branches of the created-member and sibling logic. A card that cites the member its own Create handle names (creator equal to the card number, the issue's case), a pending use of a member a forthcoming card creates, a match through the parsed ID when the spelling differs from the handle body, and an owner that is no type are all untested.
    fix: "Add rows to TestValidateFormat_MissingMemberFindings, plus one through ValidateDispatch with a forthcoming set. The first row's card both Creates `plan:sub#Thing.Fresh` and Edits `sub#Thing.Fresh`, and wants no finding. The second row's card 2 is forthcoming and Creates the member, card 3 Uses it, and it wants no finding. The third row's Uses spelling differs from the handle body but its parsed ID is the same, and it wants no finding. The fourth row Uses `sub#Foo.X`, where Foo is a func, and wants a glyph-not-found detail with no members list."
  - id: F7
    severity: LOW
    class: scope
    origin: lens:code-scope
    location: internal/websterengine/rebaseline.go:371
    summary: The board entry asks that a rebaseline touching a dead batch's cards re-arm one recovery. The diff also records unrendered amendments on terminal failed and stuck batches, which gives those batches an uncounted recovery and lets them skip the cap.
    fix: "Either limit the AmendedCards recording at rebaseline.go:371-378 for a terminal batch to bs.Status == DigestStatusDead (keep the in-flight case), and update the Rebaseline doc at rebaseline.go:220-224 and internal/websterengine/doc.go:106-108 to match; or keep the wider re-arm and say in _lym/landing/summary.md's Design decisions why failed and stuck batches get it too, so the parent can record it."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/runplan/state.go:101
    summary: The AmendedCard type doc says one card of an in-flight batch, and the BatchState.AmendedCards field doc at state.go:123 says accepted while the batch was in flight. Both are stale now that a rebaseline also records amendments on terminal failed, dead and stuck batches.
    fix: "Reword the AmendedCard doc at internal/runplan/state.go:101 to say it is one card of a begun, not-done batch that a rebaseline accepted an edit to. Reword the AmendedCards doc at :123-125 to say the edit was accepted while the batch was in flight or terminal failed, dead or stuck, and that the next recovery renders it."
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/loom/loom-rubric-plan-review.md:31
    summary: The added rubric line tells a fixer that a Card Index edit is allowed once a batch has begun. On a run with no recorded frame and no baseline copy, the gate refuses a Card Index edit too.
    fix: "Extend the line, for example: `a fix edits only its Card Index, and the gate refuses any other edit to it; on a run that recorded no overview frame the gate refuses every edit to it, so a card-set change needs a fresh run`."
  - id: F10
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:59
    summary: The Merge danger blast radius names only webster. The diff also changes the planglyph glyph-not-found policy and the loomshed plan gate, which every Plan-Write and plan-review gate runs, Webster or not.
    fix: "Change the blast radius line in _lym/landing/summary.md to name webster, the plan gate (loomshed ValidatePlan) and planglyph's glyph status findings, which every plan-writing and plan-fixing run goes through."
---

### [BLOCKING] A resumed recovery never gets the parent's answer, and nothing bounds the asking loop

**Location:** `internal/websterengine/recoverbatch.go#recoverSpawn`, with `awaitTerminal` at `:892` and the Master rung at `contracts/stencils/webster/webster-template-master.md:140-141`

**Issue:** On a dead/asking turn end with a message, `awaitTerminal` (`recoverbatch.go:872-880`) records a waiting digest and stops the strand (`:892`).
The next `recover-batch` spawns a fresh strand (`RecoverSpawnOrAttach`, `:511`, `:520`) that costs no recovery (`:385`).
That spawn's prompt (`:317`) is built from `prevDigest`, `failureDigestBlock(prior)` and the tree.
`failureDigestBlock` renders only a failed digest's reasons and the amended cards (`:168-182`).
No input carries `prior.Digest.Need`, and the verb has no flag for an answer.
The Master rung says to resume "when your parent's message says the step is taken", which fits a NEEDS_PARENT step but not an answer to a question.
So the parent's answer reaches nothing that works the batch.
A fresh strand that hits the same uncertainty asks again, and each wait is uncounted.
The board entry asks to "route the question to the parent", and a route whose answer never returns does not resolve the question.
The entry's goal of no manual intervention also fails, since the loop can run without end.

### [MEDIUM] Every message-bearing turn end is treated as a question

**Location:** `internal/websterengine/recoverbatch.go:872`

**Issue:** `Classify` labels as dead/asking every turn end with no report (`classify.go:43-44`).
The new branch turns that into waiting whenever `LastTurnMessage` returns a non-empty message.
A Claude Stop event carries `last_assistant_message` on almost every turn end, so in practice no recovery turn end stays dead/asking.
A recovery that forgets its report and ends on a closing summary used to fall under `RecoveryRetry`'s commit-and-count rule.
Now it waits on the parent, with that summary quoted as a question.
The entry asks only that the agent's last message go into the halt note, and that a question reach the parent.

### [MEDIUM] The `<dir>_test` sibling listing fix has no regression row

**Location:** `internal/planglyph/planglyph_test.go#TestValidateFormat_MissingMemberFindings`

**Issue:** Commit 4cede865 fixes a missing member of an external-test unit that failed plan validation as quarry unavailable.
It adds the `GoPackageDir` fallback at `internal/planglyph/repo.go:253-257`, but no test row reaches it.
The only fixture is `sub/a.go`.

### [LOW] A LastTurnMessage error leaves the recovery strand live

**Location:** `internal/websterengine/recoverbatch.go:873`

**Issue:** The error return at `:874-876` comes before `StopStrand` at `:892`.
The doc at `:805` promises that every terminal digest removes the strand, even when the call then refuses.

### [LOW] TestLastTurnMessage misses what the offset clamp and the post-offset parse do

**Location:** `internal/gatefixer/strand_test.go#TestLastTurnMessage`

**Issue:** Negating `offset > int64(len(data))` at `strand.go:43` survives, because every fake ignores `data` except the before-offset row, which wants `""` either way.

### [LOW] Untested created-member and sibling branches

**Location:** `internal/planglyph/planglyph_test.go#TestValidateFormat_MissingMemberFindings`

**Issue:** No row covers the `creator <= c.Number` equality boundary (`resolve.go:309`), the forthcoming-card mapping to 0 (`create.go:31-33`), the `r.ID` match (`resolve.go:331`) or the `NotATypeError` branch (`repo.go:239-242`).

### [LOW] The re-arm reaches failed and stuck batches

**Location:** `internal/websterengine/rebaseline.go:371`

**Issue:** The board entry asks only for a dead batch to be re-armed.
The recording at `:371-378` also covers terminal failed and stuck batches.
Through `rendersNewAmendment` (`recoverbatch.go:385`, `:520`; `runlevel.go:1191`; `beginbatch.go:226`; `recordbatch.go:177`), each accepted edit then grants them an uncounted recovery past the cap.

### [LOW] Stale AmendedCards docs

**Location:** `internal/runplan/state.go:101`

**Issue:** The type and field docs (`:101`, `:123`) still limit amendments to in-flight batches.

### [LOW] Plan review rubric line misstates what the gate allows on an unframed run

**Location:** `contracts/stencils/loom/loom-rubric-plan-review.md:31`

**Issue:** `unframedOverviewFindings` (`internal/loomshed/gates.go`) refuses every edit to the recorded overview when the run has no frame, Card Index edits included.
The new rubric line says a fix may edit the Card Index.

### [LOW] Merge danger blast radius names less than the diff can break

**Location:** `_lym/landing/summary.md:59`

**Issue:** The `glyph-not-found` changes in `internal/planglyph/resolve.go` and `planglyph.go` run in every plan gate.
The changed `overview-frame-edited` check in `internal/loomshed/gates.go` does too.
Both reach well beyond webster.

## Rejected

- Sibling listing turning a misspelled owner into a hard error: quarry's `expand` returns not_found with a nil error for a missing type (quarry v0.4.0 `internal/engine/expand.go:247-252`), so `siblingMembers` lists nothing and the finding stands as `glyph-not-found`.
- `DetectDrift` passing nil created and members: it judges only the glyphs a repair introduced, which name symbols that already exist (`drift.go:251-271`). A plan the gate accepts is not refused later on that path.
- Mutation survivor at `internal/planparser/targetdirs.go:307`: the line is a rename only, and its behaviour is not this diff's.
- The changed `TestBeginBatch_Refusals` expectation: it follows the intended re-arm, and commit dfaec51 explains it. It is not a loosened check.
- Merge danger door for the persisted AmendedCards meaning: an older binary already exempts an unrendered amendment at every cap site (`runlevel.go:1191`, `beginbatch.go:226`, `recoverbatch.go:520` predate the diff), so a revert reads the new records without harm.

## Focus departures

- Focus 2 asks for every reader of AmendedCards, Recoveries and Terminal, status and reset included. I read the cap, retry, dead-end, begin-batch, record-batch and failBatch readers. I did not read the reset or status verbs' code, so their handling of a terminal record holding an unrendered amendment is unjudged.
