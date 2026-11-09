## Goal

lym adopt --here behind the lym s shortcut, and lym adopt --archives lists the old owner's archives

Takes: adopt-worker-record's worker, record, follow and resume; adopt-owner-handover's archive-owner tag name function
Delivers: lym adopt --here with its owned_here outcome and record take-up; the s shortcut as adopt --here --no-orch, beam start, reed attach; lym adopt --archives over a fabric read of archive-owner tags against each repository's origin branch
Exports: the fabric archive-owner tag read

## Scope

The part changes these packages:

- internal/adoptcli (about 450 changed lines)
- internal/shortcutcli (about 10 changed lines)
- internal/fabricengine (about 150 changed lines)

## Decisions

### D10 — `lym adopt --here` and the `lym s` shortcut

`lym adopt --here` adopts the subject of the worktree it runs in: `root` in the prime, the beam in a beam.
It first reads the adopt record.
An unfinished record takes the worktree up when its `check` step is not done and its resolved subject set lists the worktree's subject, or a subject the worktree reads its owner from through `hubgeom.ParentOf`.
`--here` then takes the record's request as its own, whoever the owner reads as, so under D1 and D2 it follows a live worker or resumes the record.
A record past `check` that stopped on a held-back project takes up no worktree, so its other subjects reach `beam start`.
With no record taking the worktree up, it pulls Bolt through fabric and reads the subject's owner from it; with origin unreachable it reads the local Bolt.
When no other machine owns the subject, it ends ok with the outcome `owned_here`, spawning no worker and writing no record.
Otherwise it is `lym adopt <subject>`, under every rule of D1 and D2.
In any other worktree it is a no-op that ends ok.
`--here` with a subject, `root` or `--all` is refused.
The `s` shortcut in `internal/shortcutcli/shortcuts.yaml` becomes `adopt --here --no-orch`, then `beam start`, then `reed attach`; `beam start` starts the orch.
A shortcut stops at the first argv that exits non-zero, so a stopped adopt starts nothing.

Rationale: adopt then starts from a plain shell on the machine taking over, never from the orch.
An owned-here `lym s` never meets another subject's unfinished record.

Bound: every `lym s` with no record taking its worktree up pays one Bolt pull more than today.
A Bolt pull that cannot reach origin reads the local Bolt, so a stale Bolt that reads this machine as owner skips the adopt.
`beam start` then judges the owner on the local Bolt too, its prepare only warning that origin is unreachable, so the beam starts here as `lym beam start` does today;
the next Bolt pull that reaches origin hands the subject to D11's old-owner path.

### D12 — `lym adopt --archives` lists the old owner's archives

`lym adopt --archives <subject>...` reads, through fabric, the `archive-owner/` tags D11 writes for each slug under the named subjects, and lists those whose commit the slug's branch on origin does not contain.
Each repository's tags are read from that repository's origin and compared with that repository's branch of the slug: the code branch for code tags, the weft branch for weft tags.
Teardown and `--drop-local` archives carry the `archive/` prefix, so it lists none of them.
It lists the tags per slug, each with its tag and `git log <tag>` to look at it, runs no step, and reads or writes no record.
The adopt's end envelope names it, since the old owner archives only after this adopt's owner write reaches it.
It brings nothing in.

Rationale: the archives appear after the adopt that triggers them, so the new owner needs a read it can repeat.

## Constraints

- Each step is safe to run again: a re-run adds only missing pairs, pauses only slugs the record does not list as paused, writes the same owners, judges the deploy again, starts beam steps that leave a live pair alone, and starts no second live check.
- The adopt record and `adopt.lock` are never-tracked, under `fabricengine.AdoptScratchDir` (`PATTERN-lym-files`).
  The record is machine-local by nature: it covers only this machine's watcher and dobby.
- Git runs only in `internal/fabricengine` (`PATTERN-fabric-boundary`); the dobby's Bolt pull, the archive and the archive read are fabric calls.

## Auto-mode assumptions

- "No orch start when adopt was run from an orch session" means no orch start for any subject of that adopt.
- An unstarted released task of a subject is an adopted run for D5, since the dobby would otherwise start it.
- The old owner pauses its runs before it archives, the reverse of the board's order, so no pass starts while it archives.

## Open risks

## Acceptance criteria

- D10: `lym adopt --here` in a worktree its machine owns ends `owned_here` with no worker and no record, even beside another subject's unfinished record; in a foreign-owned prime it adopts `root`, and in a beam the beam.
- D10: `lym s` with an adopt that stops runs no `beam start`.
- D10: `lym s` after a worker killed at `deploy` resumes at `deploy` and runs no `beam start` until the adopt ends.
- D10: `lym s` in a beam with no owner of its own, during an unfinished `lym adopt root`, follows or resumes that adopt.
- D10: `lym s` on a subject that reached `check`, beside a project the same record held back, ends `owned_here` and runs `beam start`.
- D12: `lym adopt --archives <subject>` lists the `archive-owner/` tags whose commit the slug's branch on origin lacks, in each repository, lists no teardown archive, and writes no record.
