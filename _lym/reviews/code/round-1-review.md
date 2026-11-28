---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: internal/landingshed/finalize.go:1244
    summary: The sweep's search method misses whole phrasings, so production comments still name keys bare in files the diff touched and in files it did not, against the Goal's every Go comment and PATTERN-config-key-names.
    fix: "Rewrite each of these comments to the full name: internal/landingshed/finalize.go:1244 `landing.yaml's finalize_fix_review_rounds` to `landing.finalize_fix_review_rounds`; internal/landingshed/finalize_pr.go:456 `landing.yaml's squash` to `landing.squash`; internal/battencli/dobbyreads.go:357 `its dobby's max_runs` to `its dobby's beam.max_runs`; internal/hubgeom/autonomy.go:1 and internal/hubgeom/doc.go:60 ``beam.yaml's `autonomy` `` to `` `beam.autonomy` ``; internal/shuttleengine/codexengine/codexengine.go:131 `Under codex.yaml's workspace-write` to `Under a codex.sandbox_mode of workspace-write`. Then search again by phrasing rather than by name: grep production comments for `[a-z]+\\.yaml('s|`'s) `?[a-z_]+`, for `<module>'s <key>` and `the <module> <key>`, and for every one-word template key (such as `autonomy`, `squash`, `engine`, `scoper`, `ladder`, `watchdog`) in backticks or after a possessive. Rewrite each hit that names a config key, and leave a hit that names a file such as outcome.yaml or names a key generically, such as `the darn.yaml key paths`."
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/battencli/childdriverkind_test.go:16
    summary: "The Goal asks for every Go comment, but no `_test.go` comment was swept; for example `shed.yaml's driver_kind` at childdriverkind_test.go:16 and `beam.yaml's pull_request` at internal/landingshed/prreview_test.go:179 and :676 remain."
    fix: "Sweep the comments of every `_test.go` file under cmd/ and internal/ with the same method as F1. That covers internal/battencli/childdriverkind_test.go:16 to `shed.driver_kind`, internal/landingshed/prreview_test.go:179 and :676 to `beam.pull_request`, and the rest the search finds. Leave test-data literals unchanged."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/configreg/keynames_enforcement_test.go#bareKeyNames
    summary: The guard's detector and skip rules have no test of their own, so a change that makes a skip too wide passes silently; the guards in cliwire, shuttleengine and lymcwd each test their detector on synthetic input.
    fix: "Add one table-driven test in internal/configreg/keynames_enforcement_test.go that builds the index with buildKeyNameIndex and feeds literal texts to bareKeyNames, and parsed source to stringLiterals, with expected names written as literals. Rows that report: ``the `scoper` key`` reports scoper; `set code.max_bounces now` reports code.max_bounces; `gate.yaml's refuse_raw` reports refuse_raw. Rows that report nothing: `loom.code.max_bounces`; `--set max_bounces=1`; `max_bounces: 3`; `the scoper key` without backticks. On parsed source: a literal inside a `KeyName(\"loom\", \"code.max_bounces\")` call, a struct tag and an import path are not returned."
  - id: F4
    severity: LOW
    class: design
    origin: handler
    location: internal/dobbyengine/reinterpret.go:314
    summary: "The sweep renamed the retired board-entry field `pull_request` as the config key beam.pull_request at reinterpret.go:314 and :320, which changes their meaning; boardengine/store.go:256 calls these the retired board.json entry keys."
    fix: "In internal/dobbyengine/reinterpret.go, restore line 314 to `and its retired pull_request and notify_on_land entry keys while a save has not dropped them.` and line 320 to `So is an entry whose retired pull_request entry key is off, which is not carried.`"
  - id: F5
    severity: LOW
    class: consistency
    origin: handler
    location: internal/dobbyengine/reinterpret.go:25
    summary: "Two splits in reinterpret.go lost words: line 25 now reads `names the change onto the per-task lookup`, which has no verb for the move, and line 182 dropped `beam config` from `carries a hub's beam config across`."
    fix: "In internal/dobbyengine/reinterpret.go, make lines 25-26 read `// PerTaskSettingsReinterpretationID names the change that moved beam.pull_request and beam.land_notice` and `// onto the per-task lookup.`, and make line 182 read `// ReinterpretPerTaskSettings carries a hub's beam config across the retired values of beam.pull_request` with `// and beam.land_notice.` on the next line."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/configreg/keynames_enforcement_test.go#bareKeyNames
    summary: "The colon skip passes a bare key followed by a colon wherever it stands, so the common message form `<key>: <problem>` names a key bare and still passes the guard."
    fix: "In bareKeyNames, skip a name followed by a colon only when the colon is followed by a space and a value that is no prose, or when the name opens a line or follows `{` or `, ` as in a document example. Add rows to F3's table: `max_bounces: must be at least 0` reports max_bounces, and `{command: x, timeout_min: 30}` reports nothing."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/configreg/keynames_enforcement_test.go#bareKeyNameAllowlist
    summary: An allowlist entry is keyed by file and name, so any new message in that file naming the same key bare also passes, such as a bare timeout_min in gatecli/tierrun.go or gateslot/runlog.go.
    fix: "Key each allowlist entry on the literal it excuses: make the key `<file>: <name>: <literal text>` and check `allowlist.Allowed(f.Rel + \": \" + bare.name + \": \" + value)` in the test loop, updating the nine entries to name their literal."
  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md:17
    summary: The change description claims more than the diff bears out. Production comments do not all name keys in full (F1), test comments were left (F2), and a comment did change meaning (F4). The blast radius names only comments, though the diff also changes `lym config` help text and adds a tier0 guard that fails future changes.
    fix: "In _lym/landing/summary.md, after F1, F2 and F4 are fixed, state that production and test comments are swept. Drop or correct `none changed meaning` to match the code as it then stands. Change `**Blast radius:** comments` to name the comments, the `lym config` help line and the new tier0 guard on production string literals."
  - id: F9
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/configreg/keynames_enforcement_test.go#qualifiedPrefixes
    summary: "qualifiedPrefixes holds `--set \"` but not `--unset \"`, so a double-quoted --unset path argument would be reported as bare."
    fix: "Add `` `--unset \"` `` to qualifiedPrefixes in internal/configreg/keynames_enforcement_test.go, beside `` `--set \"` ``."
---

### [BLOCKING] The sweep's search method leaves production comments naming keys bare

**Location:** `internal/landingshed/finalize.go:1244`

**Issue:** The Goal asks to sweep every Go comment that names a config key, and `PATTERN-config-key-names` requires the full name in every comment.
The sweep found keys by name, so it missed phrasings it did not search for.
- `internal/landingshed/finalize.go:1244` still reads `landing.yaml's finalize_fix_review_rounds`, in a file whose line 57 the diff did sweep.
- `internal/landingshed/finalize_pr.go:456` reads `landing.yaml's squash`.
- `internal/battencli/dobbyreads.go:357` reads `its dobby's max_runs`, though the diff swept `beam.max_runs` throughout battencli.
- `internal/hubgeom/autonomy.go:1` and `internal/hubgeom/doc.go:60` read ``beam.yaml's `autonomy` ``; the diff swept `beam.autonomy` in the same package's `beammodes.go`.
- `internal/shuttleengine/codexengine/codexengine.go:131` reads `codex.yaml's workspace-write`.

These misses recur across packages and phrasings, so this is raised once against the method.

### [MEDIUM] Test-file comments were not swept

**Location:** `internal/battencli/childdriverkind_test.go:16`

**Issue:** The Goal says "every Go comment and package doc", and a `_test.go` comment is a Go comment.
The range's only changed test file is the new guard.
`childdriverkind_test.go:16` still reads `shed.yaml's driver_kind`.
`internal/landingshed/prreview_test.go:179` and `:676` still read `beam.yaml's pull_request`.
A grep over test-file comments for a few keys alone hit 11 files.
The change description narrows the work to "Production Go comments", which the entry does not license.

### [MEDIUM] The guard's detector has no test of its own

**Location:** `internal/configreg/keynames_enforcement_test.go#bareKeyNames`

**Issue:** Nothing tests the colon, `.`/`%`, `--set`/`--unset`, one-word, whole-literal, `KeyName`-call and `ast.Field` skips.
The only signal is `RequireNoStale`, which notices only when the nine allowlisted underscored names stop matching in their own files.
A regression in the one-word form or the call skip would stay green.
`internal/cliwire/bannedecl_enforcement_test.go`, `internal/shuttleengine/seam_enforcement_test.go` and `internal/lymcwd/enforcement_test.go` each hold a table that tests their detector.

### [LOW] A retired board-entry field was renamed as a config key

**Location:** `internal/dobbyengine/reinterpret.go:314`

**Issue:** `CarryBeamTaskSettings` reads a board entry's retired `pull_request` and `notify_on_land` keys.
`internal/boardengine/store.go:256` calls these "the retired board.json entry keys".
The diff rewrote line 314 to "its retired beam.pull_request and notify_on_land keys" and line 320 to "an entry whose retired beam.pull_request is off".
Read that way, a reader takes them for the live beam config key.
`notify_on_land` beside it stays bare, which shows the two are fields of the same kind.

### [LOW] Two split comments in reinterpret.go lost words

**Location:** `internal/dobbyengine/reinterpret.go:25`

**Issue:** Line 25 reads "names the change onto the per-task lookup", and line 26 "It moved beam.pull_request and beam.land_notice."
The first sentence has no verb for the move.
Line 182 changed "carries a hub's beam config across" to "carries a hub across", which drops what is carried.

### [LOW] The colon skip lets a `<key>: <problem>` message pass

**Location:** `internal/configreg/keynames_enforcement_test.go#bareKeyNames`

**Issue:** Any bare name followed by a colon is skipped, after backticks, quotes and backslashes are trimmed.
A message written in the common Go form `"max_bounces: must be at least 0"` names a key bare, yet passes.
No live instance exists today, so this is a hole in the guard rather than a missed message.

### [LOW] Allowlist entries excuse a whole file, not a literal

**Location:** `internal/configreg/keynames_enforcement_test.go#bareKeyNameAllowlist`

**Issue:** `allowlist.Allowed(f.Rel + ": " + bare.name)` lets every literal in the file that names the key pass.
A new message naming `timeout_min` bare in `internal/gatecli/tierrun.go` or `internal/gateslot/runlog.go` would go unreported.
The same holds for `driver_kind` in `internal/battencli/keynames.go`.

### [LOW] The change description overstates coverage and understates the blast radius

**Location:** `_lym/landing/summary.md:17`

**Issue:** The description says production comments now name keys in full, but F1 shows they do not.
It says "none changed meaning", but F4 shows one did.
The Merge danger blast radius is "comments", yet the diff also changes the `lym config` help text.
It also adds a tier0 guard that fails any later change writing a bare key into a string literal.
The door, `two-way`, holds: no record format, setting meaning or data changes.

### [NIT] `--unset "` is missing from the qualified prefixes

**Location:** `internal/configreg/keynames_enforcement_test.go#qualifiedPrefixes`

**Issue:** The slice holds `--set "` but not `--unset "`.
A double-quoted `--unset` path argument would be reported as bare, which is a false failure.

## Rejected

- `PATTERN.md`'s `(test)` on `PATTERN-config-key-names` (focus entry 6): `internal/pattern/doc.go:48` defines `(test)` as a rule a test enforces. The guard enforces the message and help-text half of the entry. Other tagged entries are also enforced only in part, and the entry keeps the shape `internal/pattern/check.go` accepts, so the tag is not a defect.
- Scope of the guard, the help-text change and the `PATTERN.md` edit (focus entry 5): the imported issue's "What to do" asks for the guard. The help literal is the one bare literal the guard reports, so the guard forces that change, and the tag records the guard. G2's "no user-visible text" governs the comment sweep.
- `drift.capture.timeout_min` and `drift.compare.timeout_min` in `internal/driftengine/capture.go` and `compare.go`: the change description says a drift step's `timeout_min` field stays bare, but the drift template's own comments write `drift.capture.dir` the same way. So the full name is valid, and the mismatch is too small to raise.
- `gate.tiers.<tier>.slots` and `gate.tiers.<tier>.parallel` in `internal/gatecli/doc.go`: `internal/gateslot/template.yaml:10` lists both as tier fields, so the names are right.
- Code-affected-set lens: it raised no findings, and none was rejected.
