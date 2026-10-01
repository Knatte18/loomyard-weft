---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/websterengine/audit.go:558
    summary: readOnlySegment keys the allowlist on filepath.Base, so a path-spelled program such as ./cat or /tmp/x/grep classes as a known read-only program and fails open
  - id: F2
    severity: MEDIUM
    location: internal/websterengine/audit.go:609
    summary: an allowlisted git subcommand still runs an arbitrary command through a -c or --config-env value (core.fsmonitor, diff.external) or through git grep -O/--open-files-in-pager, and the classifier reads it as read-only
  - id: F3
    severity: MEDIUM
    location: internal/websterengine/audit.go:284
    summary: loop and conditional syntax (for headers, fi, done, [[, read) is not recognised, so a read-only loop such as for f in FABRIC/*; do cat $f; done now fails closed as correctness and fails the batch
  - id: F4
    severity: LOW
    location: internal/websterengine/audit.go:563
    summary: wrapper option skipping misreads a value word as the wrapped program (sudo -u cat rm x, env -u cat rm x, xargs --arg-file cat rm x), and bash scans for -c past its script operand (bash s.sh -c true), all reading a mutating command as read-only
  - id: F5
    severity: NIT
    location: internal/shedadapters/doc.go:39
    summary: new and changed comments in cards 44 and 46 break at fixed columns or list commas, and a changed ClassifyViolation line and the refusal-spec webster sentence keep a semicolon-joined clause on one line, against the Comment Line-Break Convention and the repo's semantic line-break rule
---

Range: `product.parent` is `main` in `_lyx/shed/refusal-audit/status.json`;
reviewed `git diff $(git merge-base main HEAD)..HEAD` (merge base `fafb660c3`), concentrating on cards 44–46 and `8c4b50f2b` per the focus directive.
`go build ./...`, `go vet ./...`, `go vet -tags smoke ./...`, `go test ./...` and `go test -tags integration ./...` all pass at HEAD.

The classifier claims below were confirmed with a throwaway probe test injected through `go test -overlay` (no file in the tree touched), printing `mutatingCommand` for each command.

Settled focus questions with no finding:

- `mutatingCommand`'s empty-command branch: both audits emit a fabric-reference finding only when `fabricRef.Matches(cmd)` is true, and `RefScanner.Matches` needs a fabric path or spelling in the text, so an empty Command never reaches `ClassifyViolation` in practice.
  A comment-only command (`# /fabric/x`) has zero segments and now classes as correctness, which is the fail-closed rule the card states and is negligible in real transcripts.
  The `8c4b50f2b` row change touches only the synthetic `AuditViolation{Class: ClassFabricReference}` row; not a defect.
- Card 44 tests: every row the card names is present with its stated class, every `lyx FABRICREF sync` fixture became `cat FABRICREF/webster/state.json` and still reaches the policy path, and `TestRecordBatch_UnlistedWriterFabricReferenceFailsBatch` asserts a terminal failed batch, `ErrBatchFailed`, one archived report and the `%q`-quoted command in the digest reasons.
  `mutatingSegment`, `editsInPlace` and `mutatingTools` have no remaining reference.
- Card 46: `err.Error()` ends in `then run "lyx webster accept-audit"`, so appending `, then re-step the <row> row` leaves one trailing way-forward clause, and the refusal-spec row quotes it as emitted.
  `loomshed`'s `websterProducer` passes a non-Done outcome straight through, and the recipe's `Webster` row has no `on_stuck`, so the run persists `blocked`;
  the two new tests prove the Stuck mapping and the blocked persist.
- Card 45: the new test clears `WEFT_SKIP_GIT`, stubs `openFabric` with a sentinel, asserts a non-zero exit, the message, the sentinel text and `lyx fabric commit`, and re-reads `state.json` for no pending finding and an `accepted` disposition;
  it passes under `-tags integration`.
  The refusal-spec row matches the card's trigger wording.

### [MEDIUM] Path-spelled program passes the allowlist

**Location:** internal/websterengine/audit.go:558
**Issue:** `prog := filepath.Base(words[0])` reduces `./cat`, `/tmp/evil/grep` or `./git` to an allowlisted name, so `./cat /fabric/x` classes as policy.
A path-spelled program is an unknown program, typically a script in the worktree, which card 44's intent says must be mutating.
**Fix:** accept a program spelled with a path only when its directory is a system bin directory (`/bin`, `/usr/bin`, `/usr/local/bin`);
any other path-spelled program is mutating.
Add correctness rows for `./cat` and a non-system absolute path, and a policy row for `/usr/bin/cat`.

### [MEDIUM] Allowlisted git subcommands run commands through options

**Location:** internal/websterengine/audit.go:609
**Issue:** `git -c core.fsmonitor='rm /fabric/x' -C /fabric status` and `git -c diff.external=./x -C /fabric diff` run an arbitrary program, as does `git grep -O<cmd>` / `--open-files-in-pager=<cmd>`, yet each classes as read-only because only `--output` is scanned.
**Fix:** treat a `-c`/`--config-env` global option and a `grep` subcommand carrying `-O`/`--open-files-in-pager` as mutating, with correctness rows for each.

### [MEDIUM] Shell loop and conditional syntax fails closed

**Location:** internal/websterengine/audit.go:284
**Issue:** `shellKeywords` lists the openers but not `fi` or `done`, and a `for NAME in WORDS` header, `[[ ... ]]` and `read` are not recognised.
Before card 44 an unknown word was non-mutating, so the gap was harmless;
under the allowlist, `for f in FABRIC/*; do cat "$f"; done`, `if test -f FABRIC/x; then cat FABRIC/x; fi`, `while read l; do echo $l; done < FABRIC/x` and `[[ -f FABRIC/x ]] && cat FABRIC/x` all class as correctness and fail the batch, though every program in them is read-only.
**Fix:** add `fi` and `done` to `shellKeywords`, treat a `for`/`select` header and a `[[` test as read-only (their substitutions are already recursed separately), and add the `read` builtin to `readOnlyTools`.
Add policy rows for each shape.

### [LOW] Wrapper and shell option parsing misreads operands

**Location:** internal/websterengine/audit.go:563
**Issue:** the wrapper branch skips only words starting with `-`, so an option's separate value becomes the wrapped program: `sudo -u cat rm x` and `env -u cat rm x` read `cat` and class as read-only.
`xargsValueOptions` lacks GNU's long forms, so `xargs --arg-file cat rm x` does the same.
The `bash` branch scans every argument for `-c`, so `bash s.sh -c true` judges `true` while bash runs `s.sh`.
`timeout` is safe: a misread value leaves a numeric duration in program position, which fails closed.
**Fix:** give each wrapper its own value-option set and skip each value, add the long xargs value options, and stop the `bash` scan at the first operand, skipping the values of `-o`/`-O`/`+o`/`+O`/`--rcfile`/`--init-file`.
Add correctness rows for the misread shapes.

### [NIT] New comments miss the semantic line-break convention

**Location:** internal/shedadapters/doc.go:39
**Issue:** the WebsterProducer bullet in `shedadapters/doc.go:39-42` and the inline comment at `internal/shedadapters/webster.go:94` are fixed-column wrapped mid-phrase.
`readOnlySegment`'s doc (`audit.go:543`) and the `TestRecordBatch_UnlistedWriterFabricReferenceFailsBatch` doc (`recordbatch_test.go:682`) break at list commas rather than at sentence or independent-clause boundaries.
The changed `ClassifyViolation` line (`audit.go:647`) and the refusal-spec webster sentence (`refusal-spec.md:35`) keep a semicolon-joined independent clause on one line.
**Fix:** rewrite each as one sentence per line, breaking only at an independent-clause boundary.
