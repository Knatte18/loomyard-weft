# Finalize joins a landing queue, and the parent lock's holder builds, gates, bisects and lands a batch of queued runs with one gate run

`landing-queue-batch` · Task · enhancement, landing, fabric, git, cwd, loom, shed, docs, pattern, heddle · starting · Open-Window

Decision record: [Sketch](decision-landing-queue-batch.md)

Filed from the Sketch `landing-queue`.

Takes: landing-ledger-own-tip's ledger-landing writer and its Deps seam, which the leader calls for each prefix entry with that entry's own landing commit and the records commit the move mark names. D1 to D5 and D7 to D12 whole, and D6 but the ledger writer it takes. D1: Finalize.Call's lock-free pass (gateUnlocked), its lost-race loop and landUnderLock's gated-tip recheck in internal/landingshed/finalize.go go, with finalize_unlocked_test.go deleted or rewritten onto the queue; the lock-free merge through the resolver, needsGitHubOnDisk and parentguard.go's parentlock wait stay. D2: internal/landingqueue (new), its per-parent directory named in internal/lymdirs and keyed by the branch's hex as internal/parentlock/store.go keys its records, entries through internal/state under an internal/lock file lock, and the move mark. D3 and D8 in landingshed under the lock. D4: the chain verb in internal/fabricengine beside today's merge code (merge.go, mergelifecycle.go), so fabric-core-merge-landing moves it with that code, with a commit-tree call added to internal/gitrepo/merge.go beside MergeTree. D5: the detached checkout and its restore as fabric verbs; Deps.RemoveLeftovers is the seam over fabricengine's RemoveIgnoredLeftovers; the marker's path in lymdirs; the pusher's hold-off at readLocalBranch and RefusedPushStands, in internal/fabricengine/pusherreads.go and internal/runpusher's targetreads.go and parentseams.go, or wherever fabric-core-push-origin leaves those reads; the restore on resume in shedengine.(*Shed).Run before any row runs, beside refuseUnderRunLock, wired from internal/loomcli. D6: each member's own seams built from its pair path through internal/loomcli/landingdeps.go's wiring; the guarded move as a fabric verb; the backstop at every taker of the parent lock, which today is a run's Finalize and lym fabric anchor (internal/fabriccli/anchordeps.go's Acquire), since lym fabric pull takes no parent lock. D9: batch_max in landingshed's Config and template.yaml; finalize_lost_races leaves config.go, template.yaml, keynames.go, the finalize.go comment and config_test.go and joins RetiredKeys as CategoryCrossPair, and its mentions in the four contracts/recipes comments and docs/overview.md go. D11: the glossary line after `landing` in internal/heddleengine/glossary.md. D12: PATTERN.md's PATTERN-landing line, pattern/PATTERN-landing.md's new queue section, and internal/landingshed/doc.go's lock-free and lost-race paragraphs replaced.
Delivers: Ready runs on one parent land in batches of up to landing.batch_max with one gate run each, the parent's new tip the commit the gate ran on; a failed batch is bisected under the same lock and its culprit goes to its own fixer; a batch of one, a pull-request landing and a run with outcome failed or conflict land alone as today.
Exports: internal/landingqueue's entry, outcome and move-mark types with their reader and writers; fabricengine's chain-build, detached-checkout and restore, and guarded-move verbs; gitrepo's commit-tree call; landingshed's batch_max key.

The part changes these packages:

- internal/landingqueue (about 650 changed lines)
- internal/landingshed (about 2650 changed lines)
- internal/fabricengine (about 550 changed lines)
- internal/gitrepo (about 80 changed lines)
- internal/lymdirs (about 10 changed lines)
- internal/loomcli (about 450 changed lines)
- internal/shedengine (about 60 changed lines)
- internal/runpusher (about 80 changed lines)
- internal/fabriccli (about 60 changed lines)
- internal/heddleengine (about 1 changed lines)
- contracts/recipes (about 8 changed lines)
- docs (about 5 changed lines)
- PATTERN.md (about 2 changed lines)
- pattern (about 40 changed lines)
