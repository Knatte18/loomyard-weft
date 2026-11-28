# A push-ledger landing is written from a slug, tip and records commit given as arguments, so a parent ahead by several runs' landing commits reads as owed tips

`landing-ledger-own-tip` · Task · enhancement, loom, landing · running · Code-Burler-Fix

Decision record: [Sketch](decision-landing-ledger-own-tip.md)

Filed from the Sketch `landing-queue`.

Takes: D6's ledger steps, as the Constraints' push-ledger bullet words them: the push-ledger landing write in internal/loomcli/landingdeps.go (landLocally and recordHeldLanding, calling runpusher.RecordLanding with a pushledger.Landing) takes the run's slug, the landing tip and the records commit as arguments instead of reading them from the calling run's worktree, the records commit picked as oldestHeldRecords picks it today; landingshed's Deps.LandLocally takes the run's own landing commit as its tip, which a lone run's Finalize (internal/landingshed/finalize.go, its LandLocally call) passes, and which for a lone run is the commit it lands today; on a run whose ledger landing stands, LandLocally rewrites it with the same tip, keeps its records commit while origin lacks it and the run's new records commit descends from it, and still commits the run's status records and marks its pair's push. No queue, no batch and no leader: a run's Finalize still writes only its own landing.
Delivers: One push-ledger landing writer that takes another run's slug, tip and records commit, with today's single-run landing unchanged; a parent ahead of origin by a chain of runs' landing commits, each the tip of its own run's ledger landing, reads as ahead by owed tips alone.
Exports: Fixed for landing-queue-batch: the loomcli writer of one push-ledger landing taking the parent branch, the run's slug, its landing commit and its records commit, and the landingshed Deps seam a Finalize reaches it through; the plan names both and the change description lists their signatures.

The part changes these packages:

- internal/loomcli (about 220 changed lines)
- internal/landingshed (about 30 changed lines)
