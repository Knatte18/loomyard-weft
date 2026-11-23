# Fix five fabric faults the one-repository hardener could not raise

The one-repository hardener's fourth round recorded five faults as observations, outside its incremental scope.
This change fixes each.

## O1 — a killed clone names its way forward

A `lym fabric clone` killed after it made the records store and before `_bolt` left a hub that `lym fabric reconcile` failed on with a raw `gitrepo: … not a git repository` error,
while a plain re-clone answered `hub already exists at …` with no way forward.

- `fabricengine.RefuseHubWithoutBolt` refuses a hub whose `_bolt` holds no repository.
  The refusal names the clone that replaces it, `lym fabric clone <origin URLs> --into <parent of the hub> --reset`, with `--single-repo` and the code store's URL alone for a one-repository hub.
  The reconcile verb calls it before its Bolt pull.
- `CloneHub`'s three "hub already exists" refusals name the same clone re-run with `--reset`.
- `reconcile_missingbolt_integration_test.go` removes `_bolt` from a hubforge one-repository hub and asserts both messages.

## O2 — fetches read the records layout first

A records store whose `remote.origin.fetch` was unset let `lym fabric merge` stage a merge from stale tracking refs with `ok: true`.

- `resolveMergeSources` reads the records store's origin layout before it fetches, and a layout it cannot read is the verb's refusal, naming the refspecs to restore.
- `FastForwardPair`'s records pull and the two code-tip settle fetches in `codetipsettle.go` do the same.
  The fast-forward logs the gap and continues to the code side; the settles return it.
- The Bolt pull behind `lym machine name` already read the layout and is unchanged.

## O3 — the archive-tag push is named

A takeover's `DropLocal` keeps the tips it moves off under local `archive/<slug>/<sha>` tags, and a later remote teardown of the slug pushes each one origin lacks.
The parent ruled that these tags may leave the machine, so the behaviour stays and the docs now name it:

- `lym fabric remove --help` and `lym fabric cleanup --help` say that `--remote` pushes the slug's local archive tags that origin lacks before it deletes a branch.
- `DropLocal`'s doc no longer says the tags stay local.

## O4 — legacy plain archive tags are listed

An archive tag an earlier binary pushed under a one-repository origin's plain `refs/tags/` was named by no verb.

- `fabricengine.PlainArchiveTagsOnOrigin` lists plain `refs/tags/archive/*` and `refs/tags/archive-owner/*` refs on a one-repository origin.
  Each entry names the `git fetch` and the atomic `git push` that move the tag under `refs/lym/code-tags/` and delete the plain ref.
- `lym fabric cleanup --remote` reports them under `plain_archive_tags`.
- `onerepository_lifecycle_integration_test.go` pins the listing.

## O5 — a discard's re-run is a command

A killed discard's pending record said "run the command that discarded the task branch again".

- `DiscardTaskBranch` takes the discarding command, and the job and pending record carry it as the removal and cleanup kinds carry theirs.
  `lym batten cancel <slug> --remove` passes itself;
  the batten cleanup passes none, since the dobby's next tick runs the discard again.
- The way forward prints the command when the record holds one.

## Merge danger

**Door:** two-way
**Blast radius:** fabric
