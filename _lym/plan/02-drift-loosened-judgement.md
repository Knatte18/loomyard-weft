# Card 2 — drift-loosened-judgement

**Create:**
- `internal/driftengine#JudgeFile`
- `internal/driftengine#FileRuling`
- `internal/driftengine#LoosenedRuling`
- `internal/driftengine#filesFindings`
- `internal/driftengine#JudgedFile`
- `internal/driftengine#LoosenedCheck`

**Edit:**
- `internal/driftengine#JudgeRequest`
- `internal/driftengine#JudgeRequest.want`
- `internal/driftengine#Judgement`
- `internal/driftengine#JudgementWant`
- `internal/driftengine#ParseJudgement`
- `internal/driftengine#JudgedRecord`
- `internal/driftengine#CheckResult`
- `internal/driftengine#TestParseJudgement`
- `contracts/stencils/drift/drift-template-judge.md`

**Uses:**
- `internal/driftengine#TestRecords`

**Intent:** Carry D3's file rulings through the judge request, the judge's file and the stored records, in both diff modes, before any producer uses them.

The request and the judgement:

- `JudgeFile` is one listed file: `ID` (`f1`, `f2`, … in list order), `Path` and `Diff`, its diff against the review base.
- `JudgeRequest` gains `Files []JudgeFile` (`files`) and `CommitMessages []string` (`commit_messages`), both omitempty, under either mode.
- `JudgementWant` gains `FileIDs []string`, which `JudgeRequest.want` fills from `Files`.
- `Judgement` gains `Files []FileRuling` (`files`).
  `FileRuling` is `ID`, `Ruling` and `Reason` (`id`, `ruling`, `reason`) and `Checks []LoosenedRuling` (`checks`).
  `LoosenedRuling` is `Test`, `Loosened`, `Ruling`, `Cause` and `Reason` (`test`, `loosened`, `ruling`, `cause`, `reason`).
- Four new `Ruling` constants: `RulingNone` (`none`) and `RulingLoosened` (`loosened`) for a file, `RulingCaused` (`caused`) and `RulingUncaused` (`uncaused`) for a check.
- `ParseJudgement` runs `filesFindings` under both modes besides `linesFindings` or `commandFindings`.
  `filesFindings` refuses: a file id not of this spawn, one ruled twice, a listed file with no ruling;
  a file ruling other than `none` or `loosened`; an empty file reason;
  a `none` ruling carrying checks, and a `loosened` ruling carrying none;
  a check with an empty test, loosened text or reason, a check ruling other than `caused` or `uncaused`, and a `caused` check with an empty cause.
  Each finding names the file id, and the check's test where it has one, in the existing `judgement.yaml: ` error.
  These are re-prompts through `NewJudgementGate`, which hands the parser's error back unchanged, so the judge is refused back for an unruled file or an empty reason (criterion D3).

The records, read and rewritten through `internal/state` as today (`PATTERN-tolerant-records`):

- `JudgedRecord` gains `Files []JudgedFile` (`files`, omitempty).
  `JudgedFile` is `Path`, `Key`, `Ruling`, `Reason`, `Checks []LoosenedRuling`, `Strand` and `At`; `Key` is the hash card 3 computes.
- `LoosenedCheck` is `Path`, `Test`, `Loosened`, `Ruling`, `Cause` and `Reason`, one loosened check as the last check found it.
  `CheckResult` gains `Loosened []LoosenedCheck` (`loosened`, omitempty), which the verdict renders.

Tests, tier0:

- `TestParseJudgement` gains a `files` list on its valid lines and command rows, and a row per new refusal above, each asserting its message.
- The criterion that a judgement record written before this change still reads is held by `TestRecords` as it stands: its rows, `TupleAddedToJudged` among them, write `judged.json` and scope records with no `files` or `loosened` key and read them back through `ReadRecords`, and every new key is omitempty, so the card adds no row there.

`drift-template-judge.md`: Step 1 names the request's `files` and `commit_messages`, present under either mode.
Step 2 gains the file rule: rule every listed file once, `none` when its diff widens or removes no assertion, tolerance or limit in a test, otherwise `loosened` with one check per loosened assertion, naming the test and what was loosened, `caused` when a `Loosened-Check:` trailer in `commit_messages` names that check with a measured cause, copied into `cause`, and `uncaused` otherwise, a cause that is no measurement, such as "flaky", included.
The judge decides what is a test and what is loosened from the diff, with no language or path pattern (`PATTERN-code-conventions`).
Step 3's example adds the `files` block, and its rules require a non-empty reason on every file and check ruling.
The stencil's markers do not change.

**ImpactSummary:** Adds fields and types to the drift judge's request, judgement and stored records; every new key is omitempty, so existing requests, judgements and records read as before.

**Commit:** `2: drift-loosened-judgement`
