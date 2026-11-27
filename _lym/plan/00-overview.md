---
format: 5
approved: false
language: go
---

# Plan: fabric core foundation — layout and core packages, snapshot, action list, executor loop and the layout retarget

This is part 1 of the fabric core move (D9).
It creates `internal/fabriclayout`, which holds fabric's names, path constants, pure path functions, plain option and result types and the mutation record data.
It creates `internal/fabriccore`, which holds the snapshot, phase, action list and outcome types and no rule yet.
It adds `internal/testkit/fabrickit`'s snapshot builder and, in `internal/fabricengine`, the executor loop and the phased snapshot reader.
Every caller of a name that moves is retargeted in the card that moves it, so a file that uses only layout names imports `internal/fabriclayout` and no longer `internal/fabricengine` (D7).
Every verb still runs on today's code (D5).

## Card Index

1 — scankit-banned-imports — add the scankit check that fails a package importing a banned path, stdlib included
2 — layout-package-classify — create internal/fabriclayout with its leaf tests and move the path classification and ScopedPathspec into it
3 — layout-hub-paths — move the hub path names and pure path functions into the layout and retarget their callers
4 — layout-ref-names — move the branch, tag, namespace, trailer-key, flag and subject names into the layout
5 — layout-mutation-kinds — move the mutation entry and its kinds into the layout
6 — layout-mutation-record — split the record value and MutationRecord into the layout from the engine's recorder
7 — layout-origin-types — move the reconcile, parent-tip, teardown, hand-over and replay result types into the layout
8 — layout-push-types — move the push path's plain data into the layout
9 — engine-repo-side-names — replace the engine's private repository-side constants with the layout's
10 — layout-verb-results — move the mutating verbs' plain option and result types into the layout
11 — layout-read-types — move the read verbs' plain result types into the layout
12 — core-package — create internal/fabriccore with the snapshot, phase, action list and outcome types and its leaf tests
13 — fabrickit-snapshot-builder — add the shared snapshot builder kit
14 — executor-loop — add the engine's executor loop and its error-to-class mapping
15 — snapshot-reader — add the phased snapshot reader with its git fallback for non-files ref storage

## Shared Decisions

### Decision: what the layout takes

- **Decision:** `internal/fabriclayout` takes every exported name of `internal/fabricengine` that a file outside the engine names and that is one of D1's kinds:
  a name or path constant; a pure function of absolute paths and strings; an option or result type whose fields are plain data; the mutation entry, its kinds, the record value and `MutationRecord`.
  It takes the unexported constants and pure methods those need.
  A name stays in the engine when it does I/O, takes a `lymcwd` location or a type outside the layout's import set, carries a function field or a seam, or is an error type or sentinel.
  Error types stay with the verbs that return them and move with their concern's part (D9).
  `IsReadOnlyCommand` and `TaskCommitOf` stay too, though pure: each is a classifier over text, a shell command line or a commit message, not one of D1's names, and moves with its concern.
  The cards list every moved name; the gate's build proves the set closed.
- **Rationale:** D1 names those kinds and bars the rest; D7 retargets only names the layout holds.
- **Applies to:** cards 2–11

### Decision: a move retargets every caller in its own card, with no alias

- **Decision:** each `Move` card relocates its names with their doc comments and rewrites every reference, in the engine and in every production and test file outside it, to `fabriclayout.<Name>`.
  `goimports` adds the layout import and drops an `internal/fabricengine` import left unused; a comment naming `fabricengine.<Name>` for a moved name follows.
  No card leaves an alias of a moved name in the engine (D1), and no engine name is defined from a moved one: card 8 leaves the engine's private repository-side constants on their own literals, and card 9 retires them.
  Where the engine still needs an unexported helper that moves, the layout exports it under a stated name (cards 3, 7 and 8).
- **Rationale:** the retarget lands with the foundation in one change (D7, D9), and each card builds on its own.
- **Applies to:** cards 2–11

### Decision: the recorder stays `fabricengine.Mutations`

- **Decision:** the engine's recorder keeps its type name, `NewMutations` and its methods, so its many `rec *Mutations` call sites compile unchanged.
  The layout's `Mutations` is the record value a result carries; the recorder's `Snapshot` returns it.
- **Rationale:** D1 keeps the recorder that appends and logs in the engine and puts the plain record in the layout; renaming the recorder would touch every mutating verb for no behavior.
- **Applies to:** cards 5–6

### Decision: the foundation's tests shrink

- **Decision:** a test of a moved pure name moves to the layout at tier0 or is folded into one table per kind of name;
  a test whose assertion another test already holds is deleted in the card that makes it redundant (D5, D6, the added entry "The restructure cuts fabric's test code").
  Each card's commit body lists every test it deleted, naming the test that now holds its assertion or why it guarded nothing of its own, and for each test it kept or moved, the rule it alone guards.
- **Rationale:** the added entry's criterion; the deletions here are cards 2, 3, 4 and 15.
- **Applies to:** every card with a test change

### Decision: the foundation's unreached loop functions take dead-code allowlist entries

- **Decision:** `runRule`, `outcomeClassOf`, `snapshotReader.read`, `readLocalRefs` and `fabriccore.Refusal.Error` have no production caller until part 2 drives the loop.
  `cmd/lym#testOnlyCodeAllowed` admits them, each reason naming the tests that run it and part 2 as its first caller, and part 2 removes each entry as its caller lands; the guard's stale-entry check then holds the list.
- **Departure:** `PATTERN-test-only-code` admits an unreachable function only as a helper other packages' tests share.
  The record's D9 delivers the executor loop in this part and every verb's move in later parts, so no verb may call it yet (D5).
- **Applies to:** cards 14–15

### Decision: action runners land with their kinds

- **Decision:** the executor loop dispatches each action kind to a runner and registers none in this part but the built-in wait;
  each later part adds the runner of every kind its rules emit, with D6's one tier1 test per action kind.
- **Rationale:** D9's foundation proves the shape on no rule, and a runner no rule emits would be untested code.
- **Applies to:** card 14

### Decision: snapshot fields and the seam tests' second half land with their first rule

- **Decision:** the reader of card 15 reads the fields D2 names that today's reads already serve: worktrees, local refs, origin heads, anchors, the records store and pending teardowns.
  A field no rule of this part reads, such as origin tags for the teardown child's archive rules, lands with the part whose rule first reads it.
  Each flow's tier1 seam test (D6) gains its core and executor half in the part that moves that flow; this part's reader test is the snapshot half only, since no flow moves here (D5).
- **Rationale:** D9's foundation moves no verb, so no rule exists yet to run over a snapshot, and a field no rule reads would be untested.
- **Applies to:** cards 14–15

## PATTERN changes

- `PATTERN-leaf-packages` — changes: Importing into a leaf package: only closed import set; `fabriccore` and `fabriclayout` run no git and do no I/O.
- `PATTERN-lym-files` — changes: Placing a file, temp dir, lock or link: tracked under `_lym`, never-tracked under `.lym` as a cache a fresh machine rebuilds or starts without; named only by `lymdirs`; temp under `lymtemp`, lock paths via `lock`, links via `fslink`. (test)
