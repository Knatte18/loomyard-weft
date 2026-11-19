---
verdict: BLOCKING
findings:
  - id: F16
    severity: BLOCKING
    class: design
    location: _lym/plan/14-loom-harden-wiring.md:44
    summary: "Card 14's hardenLiveRun reads a gate test envelope that a passing or failing run never prints, so every ordinary live run would come back as a wrapped no-envelope error with no report written; only a refusal prints a stdout envelope."
    fix: "In card 14, change the Create glyph to `hardenLiveRun(exitCode int, stdout, stderr []byte) (hardenshed.LiveRun, error)` and rewrite the paragraph that says it reads the call's JSON envelope. The new text: passed is exit code 0; output is stderr followed by stdout, kept whole, since the summary lines, the started lines and the failure report go to stderr; Refused is the error of the last non-empty stdout line when that line parses as a JSON object with ok false and a non-empty error, which only a refusal prints, so build output on stdout never matches; every other nonzero exit is a failed run with no Refused; the returned error is for a spawn or wait failure alone. Delete the sentence that returns an output with no envelope as an error. In `TestHardenLiveArgs`, replace the envelope rows with: exit 0 with a summary line on stderr and an empty stdout reads passed; exit 1 with a failure report on stderr reads failed with no Refused; exit 1 whose last stdout line is a refusal envelope reads Refused with its error; exit 1 whose stdout holds a JSON line without ok false and error reads failed. In card 6, change the paragraph that returns a `Run` error for 'a spawn failure or an output that is no gate envelope' to 'a spawn or wait failure' alone, and leave its `TestLiveGate` rows as they are. The fix changes no target and no verify line, so no validate-plan overlay applies."
prior:
  - id: F1
    status: resolved
    answer: "Card 8's fenced line keeps every clause and adds the `EndProcess` orphans clause. The wording compacts in two places with the same meaning, 'end only through it' to 'end only via it' and 'names its binary via' to 'is named via'. The compaction is forced, since the current rule text is 274 characters and the cap `pattern.MaxEntryTextChars` is 280, so the plain addition would run to 300."
  - id: F2
    status: resolved
    answer: "Card 7 names the bares by role, `codeBare` and `recordsBare`, in `SandboxDeps` and in the `hardenshed` doc, and card 16 does the same for `hardencli`. The remaining warp and weft spellings sit in `fabricengine`, an owner-set package."
  - id: F3
    status: resolved
    answer: "Card 16 lists `pattern/PATTERN-cli-cobra.md` under Edit and adds the `hardencli` deviation bullet to the Package naming list."
  - id: F4
    status: resolved
    answer: "Settled by the added entry 'Added after Discussion (parent, 2026-11-19): Hub gate keys are operator writes after landing', which the Shared Decision `hub-gate-keys-are-a-hub-write` cites by that title and whose three `lym config gate --set` writes it names."
  - id: F5
    status: resolved
    answer: "Card 11's `LiveRed` keys on the round's `harden-commits-<round>.json`. This holds: `fixedExit` returns a counted Stuck with a nil error when the verify gate runs out, so card 9 writes the commits file. A round with no findings still runs its gates before the fix is skipped (`internal/burlerengine/fix.go`, `checkGates`), so a clean round writes a live report and is never read as red."
  - id: F6
    status: resolved
    answer: "Card 6 now carries `LiveRun.Refused` and a gate branch and test row for each of a refused `harden-live` call and a refused copies call. The envelope parsing that card 14 prescribes to fill `Refused` is wrong, and is raised as F16."
  - id: F7
    status: resolved
  - id: F8
    status: resolved
  - id: F9
    status: resolved
  - id: F10
    status: resolved
  - id: F11
    status: resolved
  - id: F12
    status: resolved
  - id: F13
    status: resolved
  - id: F14
    status: resolved
  - id: F15
    status: resolved
---

### [BLOCKING] hardenLiveRun parses an envelope that a passing or failing gate run never prints

**Location:** `_lym/plan/14-loom-harden-wiring.md:44`, with the dependent sentence in `_lym/plan/06-live-gate.md:36`.

**Issue:** Card 14 prescribes that `hardenLiveRun` reads the spawned `lym gate test --wait` call's JSON envelope, reads `ok` as passed and returns "an output that is no envelope" as an error. `internal/gatecli` prints no such envelope for a run that reaches a verdict.
A passing tier prints its summary line to stderr and exits 0.
A failing tier prints its summary line and the failure report to stderr and exits nonzero (`internal/gatecli/run.go` `report` and `printReport`).
A JSON envelope reaches stdout only on a refusal (`internal/gatecli/doc.go`: "A refusal gives 1, with a JSON error envelope on stdout"; `output.Err`) or, without `--wait`, on a slot-busy answer.
A tier's build output also passes to stdout.
So `hardenLiveRun` would return its no-envelope error for every ordinary green and red `harden-live` run.
Card 6 turns that error into a wrapped `Run` error with no live report.
The gate would then never mark a round red or green, the fixer would never be re-prompted over a failing test, and D4's live report, `LiveRed` and the auto-accept bound would have nothing to read.
The refusal branch that F6 asked for is sound in intent, and the failed-run and passed-run branches are what break it.

## Rejected

- Focus 1, card 8's `PATTERN.md` line: the new line differs from the current one by the declared `EndProcess` clause and by two meaning-preserving compactions the 280-character cap forces; every other clause is intact. Not a defect.
- Focus 2, cards 7 and 16: the non-comment code they prescribe in `internal/hardenshed` and `internal/hardencli` spells `codeBare` and `recordsBare` only. The side names `warp` and `weft` appear in the plan only in `fabricengine` glyphs and prose, an owner-set package. Not a defect.
- Focus 3, card 16 and the package-naming deviation list: card 16 edits `pattern/PATTERN-cli-cobra.md` and adds the `hardencli` bullet. Not a defect.
- Focus 4, `LiveRed` with no live report: answers true once the round's `harden-commits-<round>.json` exists and false otherwise. A clean round still runs its gates and writes a report. Not a defect.
- Focus 5, a refused call against a failed run: card 6 separates them through `LiveRun.Refused`. The way card 14 fills it is wrong, raised as F16.
- Focus 6, the three hub gate keys: no card carries the writes, and the added entry "Hub gate keys are operator writes after landing" records them as operator writes after landing. Not a defect.
