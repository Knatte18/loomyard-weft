---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    location: internal/fabriccli/fabric.go:66
    summary: clone's Use synopsis, its mirror comment, the usage refusal in clone.go and the first help example all omit --shortname, which a fresh bind requires, so the help's own first-bind example is refused
  - id: F2
    severity: LOW
    location: README.md:169
    summary: README's Getting started clone line, docs/sandbox-hub.md step 4 and SANDBOX-FABRIC-SUITE F13 still show a fresh-bind clone without --shortname, which card 2 made a refusal
  - id: F3
    severity: NIT
    location: docs/overview.md:306
    summary: the fabric bullet card 16 and card 20 edited holds about ten sentences on one line, against the repo's one-sentence-per-line Markdown rule
---

# Webster-Review round 1

Review range: `git diff $(git merge-base main HEAD)..HEAD`, with `product.parent` = `main` read from `_lyx/shed/agent-naming/status.json` and merge base `a5f0ce156`.

Evidence driven on HEAD (`8722cd627`):
`go build ./...`, `go vet ./...` and `go test ./...` pass;
`go test -tags integration` over agentname, fabricengine, fabriccli, hubforge, hubgeom, standalonegeom, reedengine, reedcli, cmd/lyx and tools/sandbox passes;
`go vet -tags smoke ./...` passes.
Card 20's Verify grep (`! git grep -nE 'lyx-code|fabric code|…|short code' -- . ':(exclude)_lyx' ':(exclude).lyx'`) finds no match, so the negated grep exits clean.
`gofmt -l` lists three files, none of them introduced by this branch's edits (`internal/shell/posix.go` is already gofmt-dirty at the merge base).

## Verdicts on the focus directive

**Cards 17–20 keep their no-behaviour-change contract.**
I normalised every changed Go line in `c0f53d5b4..HEAD` by mapping shortname back to code and compared the removed and added lines.
The only residue is wording: "short agent-name code" became "shortname", "uncoded" became "has no shortname", and the refusal texts changed to the wording the cards prescribe.
No expected value, fixture value (`tst`, `tc`, `lyt`, `sa1b2c`) or assertion shape changed, and no assertion was dropped.
The JSON keys `shortname` and `shortname_recorded`, the flag and the verb are renamed because card 18 says so.
The live hub's `_board` holds no `.lyx-code`, so nothing already recorded is orphaned by the record rename.

**Card 18's Move group: not a defect.**
All six files exist only under their new names, and each leading file comment names its new file name.
Git records no `git mv`: a commit is a snapshot, and rename detection is a diff-time similarity heuristic.
`git log --follow` follows `shortnamecommit.go` and `shortnamebinding_test.go` across the move, and loses the other four at the default 50% threshold.
`git show -M30%` pairs all six.
Card 18 is one commit carrying both the relocation and the in-place renames.
Because most lines in these short files spell "code", similarity falls to 37–45%.
That is a consequence of the card's one-commit shape, not of how the diff was produced, so there is nothing for this segment to fix.
No package or import changed, so the "import fixup" half of the check has no work.

**Rename completeness: complete, with three exceptions (F1, F2).**
The envelope keys, the help-tree listings in `cmd/lyx/helptree_test.go` and `fabriccli/cli_test.go`, the `argsarity_test.go` key, the `uncontainedwrite_test.go` allowlist key and reason, the sandbox tool's `--shortname lyt`, and both `refusal-spec.md` rows all use the new spelling.
No consumer of the old keys remains.
Plugins and stencils never named the code record, flag or verb.
F1 and F2 cover the clone examples and synopses that never carried the flag at all, which card 2 introduced, not cards 17–20.

**Refusal and help texts match cards 18 and 19.**
`reedengine/strand.go` emits `no strand name can be formed: this hub records no shortname; way forward: lyx fabric shortname <shortname> records it, then retry` and `no strand name can be formed: the told shortname is invalid: %w`.
`shortnamebinding.go`'s clone refusals and bound-weft warning, `fabriccli/shortname.go`'s verb refusals, the flag help in `fabric.go`, `add.go`'s `<shortname>:<slug>:<role>`/`<shortname>:<role>` and `remove.go`'s `(shortname:slug:driver)` read as the cards state.
`refusal-spec.md`'s two rows name `.lyx-shortname` and `lyx fabric shortname <shortname>`, matching the emitted string.

**Comment conventions in `ef3d5e971..8722cd627`: compliant.**
Every renamed symbol's doc comment opens with its new name:
`ValidateShortname`, `StandaloneShortname`, `ShortnameFileName`, `ReadShortname`, `usableShortname`, `WriteShortname`, `resolveEffectiveShortname`, `CommitShortname`, `runShortname`, `CloneOptions.Shortname`, `CloneResult.Shortname`/`ShortnameRecorded`, `RecordedShortname`, `ShortnameFound`, `TestShortname`, `sandboxShortname`, `NameShortname`, and the renamed test helpers and tests.
The changed comment lines keep one sentence per line, and the doc.go edits in card 20 follow the same rule.
The one Markdown exception is F3.

## Findings

### [LOW] F1 — clone's synopsis and first example omit the required --shortname

**Location:** `internal/fabriccli/fabric.go:63`, `:66` and `:137`, and `internal/fabriccli/clone.go:138`.

**Issue:** Card 2 made `--code`, now `--shortname`, required on a fresh bind, but `clone`'s `Use` string still reads `clone [--reset] [--subpath <rel>] [--force-bootstrap] [--into <dir>] <weft-url> [<warp-url>]`.
The same synopsis appears in the comment above it and in `runCloneWithReset`'s usage refusal, and none of the three lists the flag.
The help's first example, `lyx fabric clone --subpath backend https://github.com/user/mono-weft https://github.com/user/mono`, is the two-argument form a first-ever bind uses.
On a fresh weft, `resolveEffectiveShortname` refuses it with `a fresh clone needs the repo's shortname`.

**Fix:** Add `[--shortname <shortname>]` to the `Use` string, its mirror comment and the usage refusal.
Give the first example `--shortname mono`.

### [LOW] F2 — operator docs show a fresh-bind clone without --shortname

**Location:** `README.md:169`, `docs/sandbox-hub.md:84` and `tools/sandbox/SANDBOX-FABRIC-SUITE.md:320`.

**Issue:** README's Getting started creates a hub with `lyx fabric clone <weft-url> <warp-url>`, which a brand-new weft now refuses.
`docs/sandbox-hub.md` step 4 says the sandbox tool runs `lyx fabric clone https://github.com/Knatte18/lyx-test-weft https://github.com/Knatte18/lyx-test`, while `tools/sandbox/main.go` passes `--shortname lyt`.
Scenario F13 in the fabric suite clones a genuinely empty weft with `lyx fabric clone <empty-weft-url> <warp-url>` and says the clone reports `ok: true -- it always did`.
That run is now a fresh-bind refusal, so an agent driving the suite hits a refusal the scenario does not predict.

**Fix:** Add `--shortname <shortname>` to README's clone line.
Add `--shortname lyt` to sandbox-hub.md's step 4.
Give F13's clone a `--shortname`, and change its "always did" note so it describes a clone that took the flag.

### [NIT] F3 — the edited fabric bullet in docs/overview.md is not one sentence per line

**Location:** `docs/overview.md:306`.

**Issue:** The repo's Markdown rule in CLAUDE.md is one sentence per line, for list items in every `.md` file and not only new ones.
The `fabric` bullet holds about ten sentences on one line.
Card 16 added a sentence to it, and card 20 rewrote words in that sentence.

**Fix:** Break the bullet into one sentence per line, with continuation lines indented as the `config` bullet above it does.
Leave every sentence's wording as it is.
