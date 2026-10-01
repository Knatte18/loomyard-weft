---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/websterengine/audit.go:684
    summary: grepOpensPager matches only an --open prefix, so git grep --op=CMD and --ope=CMD, unique abbreviations git accepts and runs, still class as read-only, and its doc claims any --open abbreviation counts
  - id: F2
    severity: MEDIUM
    location: internal/websterengine/audit.go:568
    summary: readOnlySegment strips every leading NAME=value assignment unread, so GIT_EXTERNAL_DIFF=./x git diff, GIT_CONFIG_COUNT/KEY/VALUE setting core.fsmonitor, HOME=./h git status and PATH=./bin cat, directly or through env, run a program of the agent's choosing yet class as read-only
  - id: F3
    severity: LOW
    location: internal/websterengine/audit.go:672
    summary: skipOptions and the time -o check read only a whole option word, so a short cluster ending in a value option (/usr/bin/time -vo cat rm x, sudo -nu cat rm x, env -iu cat rm x, xargs -tI cat rm x) leaves the value word in program position and classes a mutating command as read-only
  - id: F4
    severity: NIT
    location: internal/websterengine/audit.go:282
    summary: new and changed Go comments across the range keep a semicolon-joined independent clause on one line, against the Comment Line-Break Convention
  - id: F5
    severity: NIT
    location: CONSTRAINTS.md:107
    summary: new and changed markdown prose in CONSTRAINTS.md and refusal-spec.md keeps a semicolon-joined independent clause, or a second sentence, on one line, against the repo's semantic line-break rule
---

Range: `product.parent` is `main` in `_lyx/shed/refusal-audit/status.json`;
reviewed `git diff $(git merge-base main HEAD)..HEAD` (merge base `fafb660c3`, HEAD `085c17adb`).
HEAD carries no commit after round 1's fixes, so the "round-2 fix" the focus names has not landed and this round judges `085c17adb`.

At HEAD, `go build ./...`, `go vet ./...`, `go vet -tags smoke ./...`, `go vet -tags integration ./...`, `go test ./...` and `go test -tags integration ./...` all pass.

Classifier claims were confirmed with a probe test injected through `go test -overlay` (no tree file touched) that prints `mutatingCommand` per command, and git behaviour in a scratch repository on git 2.53.

Per-card mechanical check: no card in the plan carries a `Rename`, `Delete` or `Move` group, and cards 44–46 carry only `Edit` and `Prosa` groups, so no type-specific check applies beyond the build and tests.
Card 44's removal of `mutatingSegment`, `editsInPlace` and `mutatingTools` leaves no reference anywhere in the tree.

### [MEDIUM] `git grep` pager abbreviations still fail open

**Location:** internal/websterengine/audit.go:684
**Issue:** `grepOpensPager` counts a long option only when it starts with `--open`.
git's option parser accepts any unique abbreviation of `--open-files-in-pager`, and `--op` is already unique: in a scratch repository, `git grep --op=./pg.sh hello` and `git grep --ope=./pg.sh hello` both ran `./pg.sh`, while `--o` exits 129 as ambiguous with `--or` and `--only-matching`.
The probe reports `mutatingCommand` false for `git -C /fabric grep --op=./pg.sh state`, `--ope=./pg.sh` and bare `--op`.
The doc comment says "so does any `--open` abbreviation", which states neither the property git has nor the one the code has.
No test row covers an abbreviated spelling.
**Fix:** treat a long option as the pager option when its name, the part before any `=`, is at least `--op` long and a prefix of `--open-files-in-pager`, so `--o`, `--or` and `--only-matching` stay read-only.
Reword the doc to say so, and add correctness rows for `--op=`, `--ope=` and bare `--op`, plus a policy row for `--or`.

### [MEDIUM] An environment assignment can make an allowlisted program run another program

**Location:** internal/websterengine/audit.go:568
**Issue:** `readOnlySegment` drops every leading `NAME=value` word before judging the program, and a bare assignment segment is read-only.
An assignment can choose what runs, by the same mechanisms round 1 closed for git's `-c` (F2) and for path-spelled programs (F1):
`GIT_EXTERNAL_DIFF=./x git -C /fabric diff` runs `./x` (confirmed on git 2.53), `GIT_CONFIG_COUNT=1 GIT_CONFIG_KEY_0=core.fsmonitor GIT_CONFIG_VALUE_0=./x git -C /fabric status` is `git -c core.fsmonitor=./x` spelled through the environment, `HOME=./h git status` loads `./h/.gitconfig`, and `PATH=./bin cat /fabric/x` or `PATH=./bin:$PATH; cat /fabric/x` runs a worktree script named `cat`.
The probe classes each of these, and the same through `env`, as read-only.
Card 44's intent is that a command that can run an unlisted program is mutating, and leaving these spellings open makes round 1's F1 and F2 fixes bypassable by moving the option into the environment.
Card 44 also names `env LC_ALL=C grep …` as a policy row and a bare assignment as read-only, so the fix must keep ordinary assignments read-only.
**Fix:** add a check for assignments that steer which program runs or which config it loads: `PATH`, `HOME`, `XDG_CONFIG_HOME`, `BASH_ENV`, `ENV`, any `LD_*` name and any `GIT_*` name other than `GIT_PAGER` (git runs no pager without a terminal).
Such an assignment makes the segment mutating, whether it prefixes a program, reaches it through `env`, or stands as a bare assignment segment.
Add correctness rows for the shapes above and keep the `env LC_ALL=C grep` policy row, plus a policy row for `GIT_PAGER=cat git log`.

### [LOW] A short option cluster ending in a value option misreads its value as the program

**Location:** internal/websterengine/audit.go:672
**Issue:** `skipOptions` consumes a value word only after an option word that is itself in the value set, and the `time` branch's `-o` check matches only a word starting with `-o` or `--o`.
getopt also takes a value for the last letter of a short cluster, so `/usr/bin/time -vo cat rm /fabric/x` writes its report to `cat` and runs `rm`, `sudo -nu cat rm /fabric/x` runs `rm` as user `cat`, `env -iu cat rm /fabric/x` unsets `cat` and runs `rm`, and `xargs -tI cat rm /fabric/x` runs `rm`;
the probe classes all four as read-only.
The shell branch already handles this shape for a cluster ending in `o`/`O`.
The fail-open needs the value word to be an allowlisted program name, so it is contrived, the same class round 1 rated LOW;
bash's `time` keyword rejects `-vo`, so the `time` instance needs `/usr/bin/time`, `command time` or `env time`.
**Fix:** in `skipOptions`, walk a single-dash cluster that is not itself in the value set letter by letter;
at the first letter whose `-<letter>` is in the set, the rest of the word is its value, or the next word is when that letter ends the cluster.
In the `time` branch, also count a single-dash cluster containing `o` as the output option.
Add correctness rows for the four shapes.

### [NIT] New Go comments keep a semicolon-joined independent clause on one line

**Location:** internal/websterengine/audit.go:282
**Issue:** CONSTRAINTS.md's Comment Line-Break Convention breaks a line at an independent-clause boundary, and these new or changed comments join one with a semicolon on a single line:
`loopHeaders` (audit.go:282), `fabricReferenceDetail` (audit.go:269), `mutatingCommand` (audit.go:321), `ClassifyViolation`'s parent-write line (audit.go:709), `pathWithin` (audit.go:761), `internal/websterengine/auditledger.go`, `rebaseline.go`, `runlevel.go` ("both stay errors"), `state.go` ("AcceptPendingAudit clears them"), `internal/battencli/goto.go` (`readStatusFile`), `internal/loomshed/batchifier.go` (`batchifierWayForward`), `internal/shedengine/doc.go` (goto admission), `internal/shedengine/goto.go` (the `Goto` doc and its admission line), `internal/shedengine/run.go` (`producers`), `internal/shedengine/shed.go` (`RunID` and both "It is told" lines), `internal/shedverbs/goto_test.go` (`TestGotoCmd_PassingPreGotoMoves`) and `internal/webstercli/recordbatch.go` ("Nothing in state changed").
Lines where the semicolon is followed by an imperative cross-reference (`; see Shed.RunID.`) or a verbless phrase are not counted.
**Fix:** break each at its semicolon, keeping the semicolon at the end of the first line.

### [NIT] New markdown prose keeps two clauses on one line

**Location:** CONSTRAINTS.md:107
**Issue:** CLAUDE.md's markdown rule asks for one sentence per line and a break at a semicolon joining an independent clause.
The changed Shed Verb-Set Invariant line (CONSTRAINTS.md:107) carries a semicolon-joined clause and a second sentence on one line, the new Refusal Way-Forward bullet "Enforcement is review discipline plus the per-row tests; there is no scan." (CONSTRAINTS.md:130) joins two clauses, and `contracts/specs/refusal-spec.md`'s "…are not in this table; a later audit adds each as its own section." does the same.
**Fix:** break each line at its semicolon and its sentence end.
