```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I attacked every bound, ordering promise and code claim in the record.

**Findings**

- **BLOCKING — D4, the refusal derived from the move table.**
  Deriving `RetiredKeys` from the move table refuses or skips-checks the new paths under an area whose old bare key has the same name: `discussion`, `raddle`, `friction`, `plan` and `parts`.
  - `configengine`'s `retiredCovering` counts a retired key as covering a path when `path == key` or the path starts with `key + "."`.
  - So a retired `discussion` covers `discussion.model`, `discussion.max_bounces` and every other new `discussion.*` path.
  - A local file or `run_settings` setting `discussion.review_ask_parent: true` would be judged as the old key: refused under a non-None category, or admitted with no template or shape check under None.
  - D4 needs a refusal that matches the old path exactly, or tells the two apart by shape the way D5 does. It should not reuse the prefix-covering retired mechanism.

- **MEDIUM — D4 against the operator's removal path in `PATTERN-config-layers`.**
  D4 refuses an old name in `lym config <module> --unset`, but the pattern names `--unset` as the operator's way to remove a key the template lacks.
  - Counterexample: this machine's walk has run and its build stamp is current.
  - A pre-upgrade branch whose own commit edits `_lym/config/loom.yaml` with `code_max_bounces: 2` then lands into a worktree.
  - The loader refuses the file, naming `loom.code.max_bounces`. `lym config loom --unset code_max_bounces` is refused too.
  - The fix: let `--unset` admit the old names, and have the load refusal name `lym config reconcile` as a way forward, since D5 says every reconcile moves the key.

- **MEDIUM — D5's claim that the reconcile carries every file layer "before any command runs".**
  The claim holds only for files that exist when the walk runs.
  - The walk runs once per build per machine; the build stamp and the reconcile lock sit in `configengine.ScratchDir(boardDir)`.
  - A flat key can arrive after the walk: a pull or merge of a tracked worktree file, a pair that a machine on an older build seeded, or a pair that `reconcileWorktree` skipped as incomplete. A skipped incomplete pair still lets the stamp be written.
  - None of these is moved until the next build, and D4 refuses them at load in the meantime.
  - The record should state this bound and give the way forward: `lym config reconcile`, or the pair reconcile at Seed-Child.

- **MEDIUM — D7, where the board rewrite runs and when its marker is written.**
  - The hub-wide reconcile runs inside a `fabricengine.BoltWrite` closure in `reconcileHubWide`. The reinterpretation markers are written in that same commit.
  - If D7's `boardengine` save runs inside that closure and takes the board write lock, it can block on itself.
  - If it runs outside, the board-config commit that holds the marker and the board-entry commit are separate. A crash between them can leave the marker committed with flat `run_settings` that nothing ever carries.
  - D7 should say that the `run_settings` rewrite commits first, that its marker is written only after it, and that the marker is written outside the BoltWrite.

- **LOW — Open risks, mixed builds.**
  A machine on an older build still accepts flat `run_settings` through `lym board upsert` after the newer machine wrote the D7 marker. Such an entry is never carried, and Seed-Child refuses it on the newer machine.
  The record's mixed-build risk covers loading only. It should add `run_settings` writes.

- **LOW — D5, an old bare area key that is set but empty.**
  An old bare area key written as `discussion:` with an empty value is null, which is not a mapping. D5 would move it to `discussion.model: null`. The record does not say what the loader does with that value.

**Claims attacked**

| Claim | Result |
|---|---|
| The hub-wide walk runs `configsync` before the reinterpretations, and that reconcile covers loom's hub base | Held: `ReconcileHubWideAt` iterates every registered module's board file, including per-worktree modules |
| D8: `FlagCodeAutoAccept` reads the moved `code.auto_accept` | Held, given that order |
| `configengine.TopLevelValue`'s one caller reads `beam.yaml` | Held: `dobbyengine/tasksetting.go` is its only caller |
| D5: the machine layer holds no loom or tapestry key | Held: no machine category in `loomengine/config.go` |
| D5: each machine's walk carries its own local files | Held: the build stamp is per machine, in the scratch dir |
| D5: the move runs before the local prune | Held: the move is inside `configsync` |
| D5: the move covers every file layer | Broken: files that arrive after the walk, and skipped incomplete pairs |
| D5: "new path wins" when one file sets both | Held |
| D5: the old bare key is told apart by shape | Held, apart from the null case above |
| D6: an untouched profile already took the new template | Held: `profiles.yaml` is seed-only with `WholeEntries`, so the merge works per profile entry |
| D6: no check refuses flat profile keys between the merge and the reinterpretation | Held: neither `configsync` nor `hubreconcile` calls `CheckLoad` or `CheckValues` |
| D4: the refusal lets through only what the table names | Broken: the prefix-covering problem in the BLOCKING finding |
| D4: `--unset` refusal | Broken: it removes the operator's way to clear the key |
| D7: rewrite once per hub, retry on failure | Broken: the write order and lock placement are not stated |
| D7: marker blocks a second carry | Held only if the marker is written after the rewrite commits |
| Open risk: a flat key a parallel run adds is caught at merge | Held: the template check in the acceptance criteria catches it |
| D9: dotted categories, open maps and `LocalDenied` | Held, apart from the D4 retired-coverage interaction |

**Tool calls**

1. Read `ReconcileHubWideAt`, the callers of `TopLevelValue` and loom's machine categories, to see whether the hub-wide reconcile covers loom's board file and whether any loom key is machine-classified. My context had only the walk's call site.
2. Read `runprofile`'s spec flags and where the build stamp and carried markers live, to judge D6's merge claim and D5's per-machine claim.
3. Checked whether `configsync` or `hubreconcile` run `CheckLoad` or `CheckValues`, to see whether anything refuses flat profile keys before the reinterpretation runs. My context lacked this.
4. Read the head of `reconcileHubWide` and looked for the board save and lock functions, for D7's lock interplay.
5. Looked for who takes `BoardWriteLockPath`, for D7's lock interplay. This did not settle whether `BoltWrite` takes it.
