# Card 12 — core-package

**Create:**
- `internal/fabriccore/doc.go`
- `internal/fabriccore#TestMain`
- `internal/fabriccore#TestLeafInvariant_AllowlistOnly`
- `internal/fabriccore#TestLeafInvariant_NoIO`
- `internal/fabriccore#Phase`
- `internal/fabriccore#PhasePreLock`
- `internal/fabriccore#PhaseUnderLock`
- `internal/fabriccore#PhaseOrigin`
- `internal/fabriccore#Store`
- `internal/fabriccore#StoreCode`
- `internal/fabriccore#StoreRecords`
- `internal/fabriccore#StoreBolt`
- `internal/fabriccore#Field`
- `internal/fabriccore#FieldLocalRefs`
- `internal/fabriccore#FieldOriginRefs`
- `internal/fabriccore#FieldWorktrees`
- `internal/fabriccore#FieldAnchors`
- `internal/fabriccore#FieldRecordsStore`
- `internal/fabriccore#FieldPendingTeardowns`
- `internal/fabriccore#Refs`
- `internal/fabriccore#StoreRefs`
- `internal/fabriccore#RecordsStore`
- `internal/fabriccore#Snapshot`
- `internal/fabriccore#ActionKind`
- `internal/fabriccore#ActionRefCreate`
- `internal/fabriccore#ActionRefMove`
- `internal/fabriccore#ActionRefDelete`
- `internal/fabriccore#ActionTag`
- `internal/fabriccore#ActionPush`
- `internal/fabriccore#ActionWorktreeAdd`
- `internal/fabriccore#ActionWorktreeRemove`
- `internal/fabriccore#ActionAnchorWrite`
- `internal/fabriccore#ActionExcludeWrite`
- `internal/fabriccore#ActionCommit`
- `internal/fabriccore#ActionPendingWrite`
- `internal/fabriccore#ActionPendingClear`
- `internal/fabriccore#ActionWait`
- `internal/fabriccore#ActionSeamCall`
- `internal/fabriccore#Action`
- `internal/fabriccore#OutcomeClass`
- `internal/fabriccore#OutcomeDone`
- `internal/fabriccore#OutcomeRejected`
- `internal/fabriccore#OutcomeAuthRefused`
- `internal/fabriccore#OutcomeTransportFailure`
- `internal/fabriccore#OutcomeDestructiveRefusal`
- `internal/fabriccore#OutcomeAnchorStop`
- `internal/fabriccore#OutcomeFailed`
- `internal/fabriccore#Outcome`
- `internal/fabriccore#ReadFailure`
- `internal/fabriccore#Refusal`
- `internal/fabriccore#Refusal.Error`
- `internal/fabriccore#Answer`
- `internal/fabriccore#Rule`
- `internal/fabriccore_test#TestRefusal_Error`

**Edit:**
- `internal/fabricvocab#owners`
- `pattern/PATTERN-fabric-boundary.md`
- `PATTERN.md`

**Extends:**
- `pattern/PATTERN-leaf-packages.md`

**Uses:**
- `internal/fabriclayout/doc.go`
- `internal/fabricvocab/doc.go`
- `internal/testkit/scankit#AssertNoImports`
- `internal/testkit/scankit#AssertImportAllowlistNoStale`

**Intent:** Create the leaf package `internal/fabriccore` (D1) with the snapshot, phase, action list and outcome types the later parts' rules read and return, and no rule yet (D9 part 1).
In code, docs and records the core's input is the "snapshot" and its output the "action list", never a "plan" (D11).

The types (`snapshot.go`, `action.go`, `answer.go`):
- `Phase` names when a snapshot part is read (D2): `PhasePreLock` before any lock, for refusals made early; `PhaseUnderLock` once the verb holds the lock it takes today; `PhaseOrigin` for a read of origin the core asks for.
- `Store` names a repository of the hub, `StoreCode`, `StoreRecords` or `StoreBolt`, each valued at the layout's repository-side name, `fabriclayout.RepoCode`, `RepoRecords` or `RepoBolt`, so the literals are declared once (`PATTERN-code-conventions`);
  `Field` names one part of a snapshot a phase reads or re-reads.
- `Refs` maps a full ref name to its commit; `StoreRefs` holds a store's `Local` and `Origin` refs and `OriginRead`, set once origin was read.
- `RecordsStore` holds `Present`, whether the hub has a records store, and the origin ref prefixes its layout maps, `Heads` and `Tags`, empty for the identity layout.
- `Snapshot` holds `Phases`, the phases read so far in order; `Stores map[Store]StoreRefs`; `Worktrees []fabriclayout.WorktreeEntry`; `Anchors`, each worktree's committed anchor marker by path; `RecordsStore`; `PendingTeardowns []fabriclayout.PendingTeardown`; `Outcomes []Outcome`, every action outcome so far;
  and `ReadFailures []ReadFailure`, each read the engine could not complete for a class a rule decides on.
  The backoff delays D3 hands the core land with part 2's first wait rule, as the overview's snapshot-fields decision says of a field no rule of this part reads.
- `ReadFailure` holds the `Phase`, the `Field`, the `Store`, the `OutcomeClass` and the error text of a read that failed as a transport failure, such as an origin read of an unreachable remote, so the rule decides what an unreachable origin means, as `ReconcileOrigin` decides it today (D3).
  A seam fact a later part needs, such as the matched secret paths, is a field that part adds, in fabric's vocabulary (D13, the record's secret-path entry).
- `ActionKind` is the closed set D3 lists: ref create, move and delete, locally or on origin; tag; push of a refspec; worktree add and remove; anchor and exclude writes; a scoped-pathspec commit; pending-record write and clear; a wait; a seam call.
  `Action` holds `Kind`, `Store`, `Origin` (the ref change is on origin), `Ref`, `SHA`, `Expect` (the lease tip), `Refspec`, `Path`, `Paths`, `Message`, `Delay` and `Seam`; a kind reads only the fields its doc names.
- `OutcomeClass` is the class the engine maps each action's result to (D3): done, rejected, auth refused, transport failure, destructive refusal, anchor stop, or failed for anything else.
  `Outcome` holds the `Action`, its `Class` and `Detail`, the error text of a failed action.
- `Refusal` holds `Reason` and `WayForward`; `Error` returns `Reason`, then `; ` and `WayForward` when it is set.
- `Answer` is the core's answer to a snapshot (D2, D3), exactly one of:
  a refusal, `Refusal *Refusal`; a phase request, `Phase` with the `Read` fields to read in it; or an action list, `Actions`, with the `Reread` fields the engine re-reads once the list ends.
  The refusal is a pointer, so a refusal whose reason is empty still refuses and never reads as done.
  An answer with none of them means the verb is done; one with more than one is a wiring fault the executor refuses (card 14).
  The doc says every refusal a rule returns names its way forward, as `PATTERN-refusal-way-forward` asks of the engine's refusals today.
- `Rule` is a verb's rule over a snapshot, the shape every later part's core function takes.

The package imports only the standard library and `internal/fabriclayout`, and runs no git, starts no process, touches no filesystem and takes no lock (D1):
- `TestLeafInvariant_AllowlistOnly` (tier0, `//lym:guard`, `//lym:watch internal/fabriccore/*.go internal/testkit/scankit/*.go`) calls `AssertImportAllowlistNoStale` with `internal/fabriclayout`, so `gitexec`, `gitrepo`, `dotgit`, `lymcwd` or any other repository import fails it.
- `TestLeafInvariant_NoIO` (tier0, same markers) calls `AssertNoImports` with the layout's ban list: `os`, `os/...`, `io/ioutil`, `io/fs`, `syscall`, `net` and `net/...`.
- `TestRefusal_Error` (tier0, table) pins the message with and without a way forward, wants as literals.
  It is the core's only tier0 test until part 2 adds the first rule's rows; the core has tier0 tests only (D6).
- `TestMain` runs under `tmuxkit.Main`.
- `doc.go` says what the package holds, its closed import set, that it holds every rule of fabric once the later parts move them, and the vocabulary of D11.

Owner sets and PATTERN:
- `internal/fabricvocab#owners` gains `internal/fabriccore`, and `pattern/PATTERN-fabric-boundary.md`'s owner set line gains `fabriccore` (the record's `PATTERN-fabric-boundary` constraint).
- `pattern/PATTERN-leaf-packages.md` gains an `internal/fabriccore` section:
  "Imports only the standard library and `internal/fabriclayout`." then
  "It runs no git, starts no process, touches no filesystem and takes no lock: no `os`, `io/ioutil`, `io/fs`, `syscall` or `net` import, nor any import under `os/` or `net/`." then
  "Bound: the scan sees imports only; a `path/filepath` call that reads the disk is held by review." then
  "It holds the snapshot, phase, action list and outcome types and fabric's rules over them, so each rule is checked at tier0."
- `PATTERN.md` carries the two declared entry lines, verbatim:

```markdown
- `PATTERN-lym-files` — Placing a file, temp dir, lock or link: tracked under `_lym`, never-tracked under `.lym` as a cache a fresh machine rebuilds or starts without; named only by `lymdirs`; temp under `lymtemp`, lock paths via `lock`, links via `fslink`. (test) — [background](pattern/PATTERN-lym-files.md)
- `PATTERN-leaf-packages` — Importing into a leaf package: only closed import set; `fabriccore` and `fabriclayout` run no git and do no I/O. — [background](pattern/PATTERN-leaf-packages.md)
```

  The `PATTERN-lym-files` line is unchanged; card 2 made its background edit.

**ImpactSummary:** A new leaf package of plain types that nothing calls yet, one owner-map entry, two PATTERN background sections and one PATTERN overview line; no existing behavior changes.

**Commit:** `12: core-package`
