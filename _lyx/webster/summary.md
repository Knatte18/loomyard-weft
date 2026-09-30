# Batten hub-ready: durable run records, driver liveness, fabric archives

All 16 batches were already `done` when this session started, with no reported deviations from the plan's declared file-ops.

- **Shed and loom groundwork (01–03):** a Stuck verdict can be marked budget-exempt and is then not counted, shedrun declares a durable drive-reports directory, and loomengine moves friction notes and archives under `_lyx/loom` while the lock stays in `.lyx`.
- **Loom records (04–08):**
  - The driver strand name moves to loomengine, and reed's no-session error gains a sentinel so batten can probe the driver.
  - The per-transition status commit stages the loom durable dir and drive reports.
  - Drive reports are written to the durable directory.
  - A new `lyx loom commit-records` verb commits and pushes the run records.
  - The driver's end-of-session command runs it before strand removal.
- **Batten inner run (09–10):** approval, driver-liveness and clock seams are added on `InnerRunDeps`. Run-Shed now watches for awaiting, resumes per approval, and waits for the driver to exit.
- **Fabric archives (11–14):** an archive helper tags and pushes the weft tip under `archive/`. `Remove`, `RemovePairBranch` and `Cleanup` archive before deleting a branch, and a sentinel marks `Remove`'s dirty-sibling refusal.
- **Batten teardown and integration (15–16):** teardown names `commit-records` and a resume, never `--force`. An end-to-end test covers awaiting, approval, resume, done, teardown, and records archived.

An earlier integration report (FAILED at `0904bc1de`) predated the later fix commit `a25c57c0d`. I re-ran the integration fork at that HEAD. The plan-level verify passed: build, unit tests, `-tags integration` tests, and `go vet -tags smoke`. The report is `status: OK`, with no deviations.
