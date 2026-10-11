---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:67
    summary: D3 and D4 send run to a driverless seed after which lyx loom run still refuses, because run needs a status file only start or step creates
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:33
    summary: D1 rests on the premise that an abandoned turn's gap ends at a prompt line, but transcripts on this machine show day-long stalls ending at assistant and tool_result lines, which D1 keeps as working time
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:42
    summary: D1's Bound misses compaction, where the summary is a turn-start user line written one to six minutes after the /compact line, so every compaction's time moves from working to idle
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:79
    summary: D3 and D4's placeholder parent branch is unbounded; a wrong value writes a seed that step's seed agreement refuses and that no named way forward clears
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:53
    summary: D2 judges a transcript by its earliest timestamped user or assistant line but does not decide the case of a transcript that has none
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:88
    summary: D4's and D5's Rationale argue against rejected alternatives that the support log already records, which the writer's stencil keeps out of Decisions
---

### [BLOCKING] The driverless seed route does not work for `lyx loom run`

**Location:** `_lyx/discussion/decision-record.md:67` (D3), D4, and the matching Acceptance criteria.

**Issue:** D3 says `lyx shed seed self --recipe loom --driver go --param parent=<parent-branch>` "seeds a driverless run that `step` or `run` then drives", and D4 puts that route into `run`'s missing-seed refusal.
The premise is false for `run`.
`lyx shed seed` writes only `seed.json`.
`loomPreRun` (`internal/loomcli/arm.go`) first stats the status file and refuses with `loom: no status file at ...; run "lyx loom start" first to bootstrap this task`, and `run`'s own help says "run never seeds a status file".
Only `start` and `step` (through `seedAndCommitBootstrap`) seed the status file.
So an agent that follows D4's refusal for `run`, seeds driverlessly and re-runs `lyx loom run` gets a second refusal that names only `lyx loom start`.
That is the failure the task exists to fix: the agent is sent to the LLM driver it did not want.
Scope's Out list ("any change to what `lyx loom step` or `lyx loom run` do") rules out making `run` seed, and the status-file refusal is not in Scope at all.

**Fix:** Correct the route in D3, D4 and the criteria: after a driverless seed, the first call is `lyx loom step`, which seeds the status file, and `run` drives only from then on.
Either name that order in `run`'s help and in `run`'s refusal remedy, or bring `loomPreRun`'s status-file refusal into Scope and give it the same driverless remedy (a refusal-spec row and a test, per `PATTERN-refusal-way-forward`).
Add an acceptance criterion that walks the route: driverless seed, then the documented next command, reaches a producer without naming `lyx loom start`.

### [BLOCKING] D1 does not catch the abandoned-turn gaps the transcripts show

**Location:** `_lyx/discussion/decision-record.md:33` (D1's Rationale and Bound), Open risks, Acceptance criteria.

**Issue:** D1's Rationale assumes that after a crash the gap ends at "the resume's prompt", and the Open risks say "no crash transcript was available to confirm".
Transcripts under `~/.claude/projects/` answer this.
In the `git-robustness` pair, session `b616a707-...` has an assistant `tool_use` line at 2026-10-08T20:15:15Z followed by its `tool_result` at 2026-10-10T04:03:17Z, and session `0f77046d-...` has a `tool_result` at 2026-10-08T20:17:01Z followed by an assistant line at 2026-10-10T04:02:12Z.
Both sessions froze at the same moment and resumed with no user line in between, about 32 hours later, which fits a suspended host, not a `--resume`.
Across transcripts since 2026-09-01 there are other gaps over 30 minutes after a non-turn-ending line that end at an assistant line or a `tool_result` (for example 33418 s, 32260 s, 8508 s, 5068 s), and two hour-long gaps that end at a `[Request interrupted by user]` line.
D1 removes none of these, because the later line is not a turn-start line.
The Goal's first item, "a crashed or abandoned turn's gap ... stops counting as working time", stays unmet for every observed long gap except those ending at a prompt.

**Fix:** Settle the abandoned-turn case on the evidence above, and record the choice in D1.
One option is to treat a gap after a non-turn-ending line as idle when it exceeds a bound set well above the longest legitimate foreground tool wait.
The cost is the long-thought and long-tool case D1's Bound wants kept, and a blocking `lyx ... await` call that waits for hours.
D1 then has to state which of those it gives up.
Whatever the choice, replace the "no crash transcript was available" risk with what the transcripts show, and add an acceptance criterion for a stall that ends at an assistant or `tool_result` line.

### [MEDIUM] Compaction becomes idle time, against D1's "changes nothing" claim

**Location:** `_lyx/discussion/decision-record.md:42` (D1's Bound), D1's Rationale.

**Issue:** D1's turn-start test makes the compaction summary line a turn-start line: a user line whose content is a string, no `tool_result`, neither an interrupt marker nor `<local-command-stdout>`.
In real transcripts that line follows the `/compact` command line by one to six minutes, and the command line does not end the turn, so today that interval counts as working.
A scan of transcripts since 2026-09-20 finds 67 compaction summaries over 30 s after their predecessor, 6243 s in total and up to 358 s each.
Under D1 all of it becomes idle.
That contradicts the Rationale ("in a normal transcript ... the rule changes nothing there") and the Bound ("mid-turn user text lines ... follow their predecessor within moments").
Compaction is model work with a token cost, and lyx itself triggers it through `lyx orch distill` and refresh cycles.

**Fix:** Decide whether compaction is working time, and say so in D1.
If it is, exclude the compaction summary line from the turn-start definition.
Either way, correct the Bound and add an acceptance criterion for a `/compact` line followed by its summary.

### [MEDIUM] The placeholder parent branch has no stated bound

**Location:** `_lyx/discussion/decision-record.md:79` (D4's Rationale), D3.

**Issue:** D3 and D4 add a route that has the agent type the parent branch by hand, and D4 justifies the placeholder only by keeping the refusal I/O-light.
They do not state what a wrong value lets through.
In the code, `seedAndCommitBootstrap` rewrites loom's seed with the recorded parent from the origin record, and `shedrun.WriteSeed` refuses a disagreeing existing seed.
So a wrong `<parent-branch>` produces a seed that every `lyx loom step` refuses at the seed stage.
The refusal spec's `seed disagrees` way forward, "keep the existing seed and drive it", leads back to the same refusal, and a second `lyx shed seed` with the right value is refused as a disagreeing seed too.
The agent is stranded unless something deletes `seed.json`, which no named verb does.
This is the attack-surface question the rubric requires answered for a new routing edge.

**Fix:** In D3 or D4, state the bound: what a wrong parent value causes and what way forward clears it.
If no current verb clears it, decide whether to name where the agent reads the right value (the pair's recorded parent), so a wrong value is unlikely, and record the stranded case under Open risks.

### [LOW] D2 leaves the no-timestamp transcript undecided

**Location:** `_lyx/discussion/decision-record.md:53` (D2).

**Issue:** D2 drops a transcript when its earliest user or assistant timestamp is before the bound.
A transcript with no timestamped user or assistant line, such as a session that was opened and never prompted, has no such timestamp.
D2 does not say whether it is kept or dropped.
`countFile` still adds one to the role's session count for it, so an older run's empty sessions would keep inflating the counted run's session count, a figure D2 lists as filtered.

**Fix:** State in D2 which way such a transcript goes; dropping it when a bound exists matches D2's intent.

### [LOW] Rejected-alternative argument sits in Decisions

**Location:** `_lyx/discussion/decision-record.md:88` (D5's last Rationale sentence) and `:79` (D4's Rationale, second clause).

**Issue:** D5 ends with "Pointing reviewers at `lyx loom validate-plan` as a pre-check is not adopted ...", and D4's Rationale argues against reading the recorded parent ("adding an origin-record read there would widen a deliberately I/O-light check").
Both are rejected alternatives, and both already stand in the support log's Rejected alternatives.
The writer's stencil says Decisions carries Decision and Rationale only, and never lists a rejected alternative.
Completeness check: neither sentence carries a requirement the plan writer needs beyond what D4 and D5 already state (the refusal reads only the seed and run list; the rubric only points at the checks).
D1's sentence on `stop_reason` null is different: it constrains the implementation and stays.

**Fix:** Delete D5's last sentence and D4's clause from "and adding an origin-record read"; the support log keeps both.

## Focus outcomes

- D2's Bound and the in-flight and finished rule: the newest-tag versus next-older-tag rule reads the same in Decisions, Open risks and the acceptance criteria.
  The Bound names the only transcripts the filter can drop, and the per-run left-out line makes a wrong drop visible in the report, so it is not a defect.
- D1's turn-start definition agrees with the criteria on interrupt-marker lines and mid-turn user text lines.
  The unbounded cases are F2 and F3.
- D3 and D4's refusal split (run and step name both routes, status, pause and goto name start only) reads the same in Decisions, Scope and the criteria.
  The route's limits are F1 and F4.
- D5 and D6 carry their requirements in the record itself: the pinned claim, the unchanged episode keys and the `doc.go` wording.
  D6's premise checks out: `noticeKey` holds condition, state and stamp, not the text, and `conditionText` returns the finding's own text without `haltText`.
- Relocation: F6.
