# A hardenshed test leaves .lock in the package directory

`hardenshed-test-stray-lock` · Task · bug, gate

## Seen

2026-11-19: Finalize of `hardener-campaign` stopped on an untracked zero-byte `internal/hardenshed/.lock`, created 18:39 during the run's gate, held by no process.
`internal/hardenshed`'s `round.go` and `live.go` take their lock through `lock.TrackedFileLockPath(path, ".lock")`; a test that passes a relative or empty record path makes that lock land in the package directory, which is the test's working directory.
The orch removed the file by hand so Finalize could go on; the next gate run recreates it.

## Idea

Find the hardenshed test that writes a record outside `t.TempDir()` and point it there.
A gate check that fails on any new untracked file in the source tree after a test run would catch the whole class.
