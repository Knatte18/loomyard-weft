# Card 12 — core-package

**Create:**
- `internal/fabriccore/doc.go`
- `plan:internal/fabriccore#TestMain` -> `func TestMain(m *testing.M)`
- `plan:internal/fabriccore#TestLeafInvariant_AllowlistOnly` -> `func TestLeafInvariant_AllowlistOnly(t *testing.T)`
- `plan:internal/fabriccore#TestLeafInvariant_NoIO` -> `func TestLeafInvariant_NoIO(t *testing.T)`
- `plan:internal/fabriccore#Phase` -> `type Phase string`
- `plan:internal/fabriccore#PhasePreLock` -> `const PhasePreLock Phase = "pre_lock"`
- `plan:internal/fabriccore#PhaseUnderLock` -> `const PhaseUnderLock Phase = "under_lock"`
- `plan:internal/fabriccore#PhaseOrigin` -> `const PhaseOrigin Phase = "origin"`
- `plan:internal/fabriccore#Store` -> `type Store string`
- `plan:internal/fabriccore#StoreCode` -> `const StoreCode Store = "code"`
- `plan:internal/fabriccore#StoreRecords` -> `const StoreRecords Store = "records"`
- `plan:internal/fabriccore#StoreBolt` -> `const StoreBolt Store = "bolt"`
- `plan:internal/fabriccore#Field` -> `type Field string`
- `plan:internal/fabriccore#FieldLocalRefs` -> `const FieldLocalRefs Field = "local_refs"`
- `plan:internal/fabriccore#FieldOriginRefs` -> `const FieldOriginRefs Field = "origin_refs"`
- `plan:internal/fabriccore#FieldWorktrees` -> `const FieldWorktrees Field = "worktrees"`
- `plan:internal/fabriccore#FieldAnchors` -> `const FieldAnchors Field = "anchors"`
- `plan:internal/fabriccore#FieldRecordsStore` -> `const FieldRecordsStore Field = "records_store"`
- `plan:internal/fabriccore#FieldPendingTeardowns` -> `const FieldPendingTeardowns Field = "pending_teardowns"`
- `plan:internal/fabriccore#Refs` -> `type Refs map[string]string`
- `plan:internal/fabriccore#StoreRefs` -> `type StoreRefs struct`
- `plan:internal/fabriccore#RecordsStore` -> `type RecordsStore struct`
- `plan:internal/fabriccore#Snapshot` -> `type Snapshot struct`
- `plan:internal/fabriccore#ActionKind` -> `type ActionKind string`
- `plan:internal/fabriccore#ActionRefCreate` -> `const ActionRefCreate ActionKind = "ref_create"`
- `plan:internal/fabriccore#ActionRefMove` -> `const ActionRefMove ActionKind = "ref_move"`
- `plan:internal/fabriccore#ActionRefDelete` -> `const ActionRefDelete ActionKind = "ref_delete"`
- `plan:internal/fabriccore#ActionTag` -> `const ActionTag ActionKind = "tag"`
- `plan:internal/fabriccore#ActionPush` -> `const ActionPush ActionKind = "push"`
- `plan:internal/fabriccore#ActionWorktreeAdd` -> `const ActionWorktreeAdd ActionKind = "worktree_add"`
- `plan:internal/fabriccore#ActionWorktreeRemove` -> `const ActionWorktreeRemove ActionKind = "worktree_remove"`
- `plan:internal/fabriccore#ActionAnchorWrite` -> `const ActionAnchorWrite ActionKind = "anchor_write"`
- `plan:internal/fabriccore#ActionExcludeWrite` -> `const ActionExcludeWrite ActionKind = "exclude_write"`
- `plan:internal/fabriccore#ActionCommit` -> `const ActionCommit ActionKind = "commit"`
- `plan:internal/fabriccore#ActionPendingWrite` -> `const ActionPendingWrite ActionKind = "pending_write"`
- `plan:internal/fabriccore#ActionPendingClear` -> `const ActionPendingClear ActionKind = "pending_clear"`
- `plan:internal/fabriccore#ActionWait` -> `const ActionWait ActionKind = "wait"`
- `plan:internal/fabriccore#ActionSeamCall` -> `const ActionSeamCall ActionKind = "seam_call"`
- `plan:internal/fabriccore#Action` -> `type Action struct`
- `plan:internal/fabriccore#OutcomeClass` -> `type OutcomeClass string`
- `plan:internal/fabriccore#OutcomeDone` -> `const OutcomeDone OutcomeClass = "done"`
- `plan:internal/fabriccore#OutcomeRejected` -> `const OutcomeRejected OutcomeClass = "rejected"`
- `plan:internal/fabriccore#OutcomeAuthRefused` -> `const OutcomeAuthRefused OutcomeClass = "auth_refused"`
- `plan:internal/fabriccore#OutcomeTransportFailure` -> `const OutcomeTransportFailure OutcomeClass = "transport_failure"`
- `plan:internal/fabriccore#OutcomeDestructiveRefusal` -> `const OutcomeDestructiveRefusal OutcomeClass = "destructive_refusal"`
- `plan:internal/fabriccore#OutcomeAnchorStop` -> `const OutcomeAnchorStop OutcomeClass = "anchor_stop"`
- `plan:internal/fabriccore#OutcomeFailed` -> `const OutcomeFailed OutcomeClass = "failed"`
- `plan:internal/fabriccore#Outcome` -> `type Outcome struct`
- `plan:internal/fabriccore#ReadFailure` -> `type ReadFailure struct`
- `plan:internal/fabriccore#Refusal` -> `type Refusal struct`
- `plan:internal/fabriccore#Refusal.Error` -> `func (r *Refusal) Error() string`
- `plan:internal/fabriccore#Answer` -> `type Answer struct`
- `plan:internal/fabriccore#Rule` -> `type Rule func(snapshot Snapshot) Answer`
- `plan:internal/fabriccore#TestRefusal_Error` -> `func TestRefusal_Error(t *testing.T)`

**Edit:**
- `internal/fabricvocab#owners`
- `pattern/PATTERN-fabric-boundary.md`
- `PATTERN.md`

**Extends:**
- `pattern/PATTERN-leaf-packages.md`

**Uses:**
- `internal/fabriclayout/doc.go`
- `internal/fabricvocab/doc.go`
- `plan:internal/testkit/scankit#AssertNoImports`
- `internal/testkit/scankit#AssertImportAllowlistNoStale`

**Intent:** Create the leaf package `internal/fabriccore` (D1) with the snapshot, phase, action list and outcome types the later parts' rules read and return, and no rule yet (D9 part 1).
In code, docs and records the core's input is the "snapshot" and its output the "action list", never a "plan" (D11).

The types (`snapshot.go`, `action.go`, `answer.go`):
- `Phase` names when a snapshot part is read (D2): `PhasePreLock` before any lock, for refusals made early; `PhaseUnderLock` once the verb holds the lock it takes today; `PhaseOrigin` for a read of origin the core asks for.
- `Store` names a repository of the hub, `StoreCode`, `StoreRecords` or `StoreBolt`, and `Field` names one part of a snapshot a phase reads or re-reads.
- `Refs` maps a full ref name to its commit; `StoreRefs` holds a store's `Local` and `Origin` refs and `OriginRead`, set once origin was read.
- `RecordsStore` holds `Present`, whether the hub has a records store, and the origin ref prefixes its layout maps, `Heads` and `Tags`, empty for the identity layout.
- `Snapshot` holds `Phases`, the phases read so far in order; `Stores map[Store]StoreRefs`; `Worktrees []fabriclayout.WorktreeEntry`; `Anchors`, each worktree's committed anchor marker by path; `RecordsStore`; `PendingTeardowns []fabriclayout.PendingTeardown`; `BackoffDelays []time.Duration`, the delays a rule's waits take (D3); `Outcomes []Outcome`, every action outcome so far;
  and `ReadFailures []ReadFailure`, each read the engine could not complete for a class a rule decides on.
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
