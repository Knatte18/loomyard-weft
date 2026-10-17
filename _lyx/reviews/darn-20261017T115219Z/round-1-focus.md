---
round: 1
focus:
  - "internal/gatecli/run.go and cli.go: does the slot wait use `shuttleengine.DefaultAwaitCap` as the single shared definition, and does the cap-passed path print an ok envelope with the holders and exit 75 on every route, including a cancelled or errored acquire?"
  - "internal/gateslot/config.go, template.yaml, pool.go and the gate spec's `RetiredKeys`: does a hub that still sets `cli_wait_sec` load cleanly and get pruned by `lyx config reconcile`, as the board entry requires, and is no reader of the retired key left anywhere in the repo?"
  - "internal/shellguard/rows.go and the claude settings golden: is the re-run sentence the same wording the await help uses, and does the golden diff carry only that change rather than unrelated churn?"
  - "Webster stencils and docs (`doc.go` files, `docs/overview.md`): is any rule other than the busy-answer description dropped or weakened, and do the docs match the code as changed?"
  - "Change description against the diff: does each claim in `_lyx/landing/summary.md` hold, including the `cmd/testtiming -resources` wait and the fixtures it says no longer write `cli_wait_sec`?"
  - "Test economy: is each added behavior (cap-passed envelope, retired-key load, shell guard sentence) covered once, and is any new assertion a duplicate of an existing test or a pin on stencil prose?"
---

Seed focus from the board entry's operator decision: shared cap, non-failure run-again answer with its own exit code and holders, prompt sentence, `cli_wait_sec` retired.
