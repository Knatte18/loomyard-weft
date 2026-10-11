# Add loom card insert/remove/restore verbs and refusal reachability rule


Plan editing had no sanctioned way to insert or remove a card.
Card files carry their number in the name, so one inserted card forced a hand rewrite of every later file and an unsanctioned `rm` of the old names.
Separately, Plan-Review never checked that a refusal a card prescribes can be reached, so an unreachable refusal passed review and surfaced only after its code was written.

Closes #508
Closes #516

## What changes

Three new `lyx loom` verbs in `internal/loomcli`:

- `insert-card <position> <slug> --intent "<intent>"` shifts every card at or above the position up by one, writes a heading-only `NN-<slug>.md` and adds its Card Index line.
  A position one past the last card appends with no shift.
- `remove-card <position>` deletes the card's file and index line and shifts every later card down by one.
- `restore-renumber` restores the plan directory from the journal an interrupted call leaves.

A moved card gets exactly four rewrites: file name prefix, `# Card N` heading, the `N: ` prefix of an explicit `**Commit:**` line, and its Card Index number.
The envelope lists each moved card, the created or removed file, and every prose line naming a moved card's old number (`card 8`).
Prose mentions are reported and never rewritten, since a mention may mean the old or the new card.
The new card's body is written afterwards with Edit or Write; until then the plan gate reports the stub's missing fields.

The Plan-Write and PR-Rework stencils and the Plan-Review rubric's `## Write surface` section each gain one sentence naming the two verbs as the only way to insert or remove a card file.
The Plan-Write stencil also gains a reachability rule: a card prescribing a refusal or guard branch names the input that reaches it after every upstream check passes, and the test that reaches it.
The Plan-Review rubric gains the matching `## Also flag` bullet.
Grouped transient raw I/O refusals are exempt, and a test that calls only the error constructor does not count as reaching a refusal.
Reachability stays a review judgment; no Go check was added.

Docs updated: `internal/loomcli/doc.go`, `internal/planparser/doc.go`, `docs/overview.md`, the numbering section of `contracts/specs/loom-plan-spec.md`, and the loom section of `contracts/specs/refusal-spec.md` (new rows, and the raw-I/O row widened to name `rename`, the plan directory's card and overview files and the renumber journal).

## Design decisions

- The format rewrite lives in `internal/planparser`, beside `SetApproved`; the cobra verbs live in `internal/loomcli`.
  The new `websterengine.LastBegunCard` reads the frozen-card bound from the run record.
- Only the current generation's card files and the Card Index are touched.
  `archive-*/`, `round-<N>/prior-generation/`, the overview frontmatter and every other overview section are never written, so the overview frame hash is unchanged and rebaseline needs no webster change.
- Both verbs refuse, before changing anything, an unparseable plan, an index/file mismatch, an out-of-range position, a position at or before the last card of the last begun batch (terminal or not), removing the only card, a slug that is not lowercase ASCII letters and digits joined by single hyphens, and an empty, whitespace-only or multi-line intent.
  Each refusal has its row in `refusal-spec.md` with a way forward.
- The verbs hold webster's state-mutation lease from the run-record read through the last apply step, so no begin-batch can slip in between the bound check and the renumber.
- Atomicity: pre-call bytes and every name the call will create go into an untracked journal under `.lyx` (path declared in `internal/lyxdirs`), written to a temporary name and renamed into place.
  A raw I/O failure mid-apply is reversed from the journal and joins the grouped transient raw-I/O row.
  If the reversal fails or the process is killed, the journal stays, both other verbs refuse to start, and `restore-renumber` is the way forward.
  `restore-renumber` only touches top-level card names and `00-overview.md`, is idempotent, and discards edits made to those files since the interrupted call.
- Each input refusal has its own sentinel (`ErrRenumberPosition`, `ErrRenumberOnlyCard`, `ErrRenumberSlug`, `ErrRenumberIntent`, each wrapping `ErrRenumberInput`) so each names its own way forward.
- The verbs never commit, approve or set `approved:`; the Plan-Burler row's commit seam commits the plan directory as before, deletions included.
- Standalone `lyx webster rebaseline` after a renumber needs `--card` for every number from the position to the last card plus one; the envelope's moved-card list gives them.

## To check by hand

- The lease test: a begin-batch started while a verb holds the state-mutation lease must wait, but whether the test observes the verb releasing the lease is not established (`lease-test-release-unobservable`, in `internal/loomcli/renumber_test.go`).
- `lyx gate test ./...` and `lyx gate test --tags integration ./...` were not run as part of this description.
- Run `insert-card` and `remove-card` against a real plan with a begun batch, and a `restore-renumber` after a killed call, to confirm the envelope and refusal wording read well.
