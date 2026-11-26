---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
---

# Round 1 fixer-report — Discussion of fabric-git-free-core

F18 and F19 are `decision`-class and went to the parent; no ruling reached this round, so they are left out.
Every premise of F1 to F17 held against the code (`go list` of `internal/fabriccli`'s imports, the shed uses in `internal/fabricengine`, `remoteteardown.go`, `add.go`, `mutation.go`, `hubworkspace.go`, `internal/impactset/graph.go`, go-git v5.19.1's `repository_extensions.go`).

## Changes

All in `_lym/discussion/decision-record.md` unless named otherwise.

- **F1** — D12 defines "above fabric" by dependency closure: a cut module, or a package outside the engine, core and layout whose closure holds `internal/fabricengine` or a cut module; that rule makes `internal/fabriccli` above fabric.
  D13 binds the engine, core and layout only, and leaves `internal/fabriccli` unbound with its imports kept as the isolation debt.
  Every seam is installed at process start by `cmd/lym/main.go`, so both detached children's entry points take installed seams.
  The D13 guard and criterion now check the production dependency closure; the `PATTERN-fabric-boundary` changes line names the three bound packages; Scope Out gains `internal/fabriccli`'s imports.
- **F2** — D13's shed bullet: shed's records cross as bytes with shed's codec behind the seam.
  The code-tip settle and takeover write go through the seam's encoder under the seam's lock path; branching rules ask the decoder for a verdict.
  Status readers return bytes, which their callers decode; takeover entries become fabric's own entry, which the decoder fills. D5's guarantee is stated per kind.
  New criterion: a test in the shed module's package pins the status file bytes before and after the cut.
- **F3** — D13's vscode bullet: an installed workspace builder that fabric calls after the verb's changes, on every exit that regenerates today.
  Fabric writes the bytes through the hub `os.Root` under the workspace lock. The builder replaces `InstallWorkspaceContent`, and `InstallPairFolderNamer` stays.
  A failing builder is a regeneration failure; with none installed, nothing is written.
- **F4** — D2 lists `remoteTeardownJob`'s handed facts, the parent's verdicts and the child's core rules.
  The pending-teardown record keeps its shape for its readers, and the in-process `holdTeardown` path follows the per-child snapshot rule.
- **F5** — D3: fabric-defined outcome classes, with the error-to-class mapping kept in the engine.
  The backoff delays reach the core in the snapshot, and a wait action is added. The archive tag push's `PushWithRetryReasons` loop stays inside one action.
  The rationale names both kept exceptions.
- **F6** — D3: a failed or refused action ends the list. D4: the inverse list covers the executed actions plus pre-creation facts, and the engine keeps the ownership tokens.
  The effects with no inverse are named, and today's best-effort rule is kept. The criteria are split and widened.
- **F7** — D4: the executor records exactly today's entries, kind for kind, and the background clause is restated to match. New criterion: the `mutations` array is unchanged.
- **F8** — D1: the layout holds the mutation entry, its kinds and `MutationRecord` as plain data, and the engine keeps the recorder.
  D7 and D9: a result type that carries a shed type moves in the part that cuts its shed import.
- **F9** — D7's rationale is narrowed to importers whose tests reach no engine importer. D10 counts the packages that left the engine's gate scope.
  Open risk 3 and Q15 in `support-log.md` now also ask about a name-only split of `internal/testkit/boardkit` and `internal/testkit/envkit`.
- **F10** — D12 admits only the kits none of whose imports is above fabric, `internal/hubforge` aside, and names examples outside that set.
  A test that uses such a kit moves or is rewritten. The guard and its criterion fail on an import of a kit outside the set.
- **F11** — D2 pins, from go-git v5.19.1's `verifyExtensions`, that `gitrepo`'s `git.Open` refuses a `refstorage` extension.
  The reader falls back to git for refs as well as worktrees whenever `dotgit` reports `ErrRefStorage`. D6 and its criterion cover both reads.
- **F12** — D13 states the meaning of an unset value for each seam.
  The run-records paths, run locks, shed's codec and the stencil judge are required, and a verb refuses before acting without them. An optional seam's nil keeps today's meaning.
  New criterion: a verb with a required seam unset refuses before acting.
- **F13** — D10 measures before and after in one session on one machine, median of two runs each, recording the load as the 2026-11-03 survey does.
  The target is now a cut of at least half against that measure, with the evidence it rests on and its gap. The D10 criterion, the auto-mode bullet and Q10 are updated.
- **F14** — D12: the core takes config values as snapshot inputs, so a core row sets them as fabric's own state, and engine tests load real config.
  This is the review's second option. The parent may want to weigh it against the operator's real-config correction (support-log Turn 5).
  The first option cannot be carried out: `configengine.Load` needs `fabricengine.ConfigSpec()` and an initialized hub, and `internal/configengine`'s closure holds `internal/lymcwd`, `internal/dotgit` and `os/exec`.
- **F15** — D9: part 1 carries only the `PATTERN-leaf-packages` entries. Every other PATTERN edit lands in the part where its text first holds, the `PATTERN-fabric-core` entry and the `PATTERN-fabric-boundary` rule line in part 7.
- **F16** — D1 names both closed import sets, and the Notes give `PATTERN-leaf-packages` a section per new package.
- **F17** — D2 names fabric's two detached children and says the background pusher is `internal/runpusher`'s process, which uses fabric's public verbs.

- **Gate repair (pattern-change-over-cap)** — the `PATTERN-fabric-boundary` changes line ran to 299 characters, over the 280 cap. It now reads "all but `fabriccli` import nothing above fabric", which keeps the same meaning in 265 characters.

`support-log.md` also gains a Round 1 entry, rejected alternatives for the shapes not taken, and Q16 for this round's auto-picks.

## Self-check

Every fork returned a report; each was saved beside this report.

- fix-consistency, MEDIUM: building the children's seams in `internal/fabriccli` extends fabric's shed knowledge. Fixed differently from where the change first stood: seams are installed at start by `cmd/lym/main.go`, and `internal/fabriccli` builds none.
- fix-consistency, MEDIUM: the required-seam refusal and the builder's nil meaning breach D5 and Scope Out. Fixed: D5 says both reach only a process that leaves a seam unset, and the `cmd/lym` criterion covers the required seams too.
- fix-consistency, LOW: other kits are also outside the admitted set. Fixed: D12 names the rule's examples and generalizes the move rule.
- fix-claims, MEDIUM: "a failed inverse action is logged" is false today. Fixed: D4 now says only a failed or refused warp-branch delete is logged.
- fix-claims, LOW: envkit is also outside the kit set. Fixed with the same D12 change.
- fix-claims, LOW: go-git's refusal under format version 0 uses another error. Fixed: D2 names both errors.
- fix-proposal, MEDIUM: fabriccli building shed seams breaks the isolation rule. Merged with the fix-consistency finding above.
- fix-proposal, MEDIUM: a takeover entry is decoded data, not a verdict. Fixed: the decoder fills fabric's own takeover entry for `ReadUpstreamTakeover`, the takeover stop and the result types.
- fix-proposal, MEDIUM: fabric's tests cannot run flows that need a required seam. Fixed: path and lock seams take a location in the test's hub, the seam tests for codec and judge flows go in the supplying module, and the engine tests their git effects as action kinds.
- fix-proposal, LOW: `holdTeardown` also runs in process in production. Fixed: D2 names both conditions.
- fix-way-forward, MEDIUM: a core row cannot reach fabric's config spec. Fixed differently: D12 now takes the review's other option for F14 (see F14 above).
- fix-way-forward, MEDIUM: required seams leave fabric's tests no way to run. Merged with the fix-proposal finding above.
- fix-way-forward, MEDIUM: measuring the before tree needs a checkout an implementer cannot make. Fixed: the close-out part asks its parent for one.
- fix-way-forward, LOW: the byte-identity test has no home. Fixed: the criterion puts it in the shed module's package.
- fix-way-forward, LOW: the `cmd/lym` test names no observation. Fixed: it runs the binary on a hub and checks the workspace file is written and the required-seam verb runs without refusal.

## Disputed

None.

## Deferred

None.

## Rulings

None reached this round.
