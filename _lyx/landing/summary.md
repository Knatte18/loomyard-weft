# Gate hygiene: PATTERN headroom, budget hints, gofmt guard, /proc probes

Two shared caps, the `PATTERN.md` overview size and the per-package test budget, blocked runs at Publish only after main was merged in.
This change stops the first from happening and makes the second a one-line fix.
It also adds a gofmt check to the test suite and folds the last test-side `/proc` copies onto `internal/testkit/tmuxkit`.

## What changes

- **Overview headroom.**
  `internal/pattern` declares a headroom line at 95% of `MaxOverviewBytes` (15564 bytes), derived by integer arithmetic.
  A pure function compares the overview at a base and at the head and fails only when the head grew and ends above the line.
  Its finding names the head size, the base size, the line and the bytes to cut to the larger of the base and the line, so a fixer is never told to cut wording the base brought in.
  A change that leaves the size unchanged or shrinks it passes at any size.
- **`lyx loom lint-pattern`.**
  A new verb in `internal/patternlint`, with the same base semantics as `lint-comments` (working tree against HEAD, or `--base <commit>` against HEAD) and the same JSON envelope.
  The webster card gates append it after `lint-comments`.
  The Webster-Burler round gate (`loomshed.NewVerifyGate`) runs the same function in-process from the branch's merge base with its parent to HEAD, before any test.
  A standalone run records no parent branch and skips it.
  `pairMergeBase` (renamed from `darnMergeBase`) now serves every pair run's wiring.
- **Overview trim.**
  `PATTERN.md` is cut to 14725 bytes.
  Detail moved into the entries' background files, six of them new; every entry keeps its name, trigger, requirement and `(test)` mark.
- **Hard-cap finding.**
  `pattern.Check`'s oversize finding now states the size, the cap and the bytes over it.
- **Test-budget messages.**
  `TestTestBudget_PackagesStayWithinBudget` names the row value to set in `cmd/lyx/testdata/test-budget.yaml`, or tells the reader to add a row with the count.
  The guard's logic and the file format are unchanged.
- **gofmt guard.**
  An untagged `//lyx:guard` test in `cmd/lyx` formats every module `.go` file in-process with `go/format.Source`, skips `testdata`, and fails with the file and the first differing line, current and formatted.
  A file that does not parse fails by name, and a scan floor keeps a broken walk from passing.
  The files it flagged are fixed here.
- **`/proc` probes.**
  `tmuxkit` gains parent-pid, process-state and comm probes, sharing stat parsing with its session probe.
  The `loomcli`, `orchcli`, `reedcli`, `pairteardown`, `shedverbs` and `gatecli` tests read `/proc` only through the kit.
  Matcher logic used by one package stays in that package.
  The tier-purity allowlist rows follow.

## Design decisions

- The headroom binds growth, not size.
  A lower cap enforced by the same `go test` would run on the merged tree too and move the collision rather than prevent it.
  The hard cap stays 16 KiB and still bounds everything the lint lets through.
- The round gate diffs from the merge base with the parent, not from the comment lint's base.
  After a Publish merge-in, the comment lint's base would count main's growth as the branch's; the merge base holds a branch to its own growth only.
  Telling the gate the reader does not switch a loom run's comment-lint base or impacted-set command.
- Publish routing is unchanged.
  A Publish verify failure still stops Stuck with the existing `lyx loom goto --to Webster-Burler` way forward.
- No PATTERN entry is added for the gofmt guard, since gofmt-clean code is the toolchain's convention and not a structural invariant.
- The verb and both gate sites call one `internal/pattern` function, per `PATTERN-gate-self-check-parity`, and its background file lists the pair.
  The "Webster-Burler verify gate exhausted" row of `refusal-spec.md` names the headroom finding as a cause.

## Check by hand

- The overview trim rewrites authoritative text.
  Compare each shortened `PATTERN.md` entry with its prior wording for a dropped requirement.
- The old-reader meaning of the merge-base reader is stated in files the card that renamed it did not target (the `loomshed`, `loomcli` and `websterengine` docs and comments).
  Grep for stale `darn`-only wording.
- A Go toolchain upgrade can change formatting and flag older files in the gofmt guard; the failure names the line, and the fix is an edit.
- `small-fixes-batch` may raise the same `cmd/lyx` budget row; a collision at Publish is fixed from the new message.
- Another branch that grows `PATTERN.md` and lands first can push the merged overview over 15000 bytes; the hard cap still guards that tree.
