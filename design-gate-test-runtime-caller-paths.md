# gate: runtime.Caller returns module-relative paths under lyx gate test, as with -trimpath

`gate-test-runtime-caller-paths` · Task · bug, gate · running · PR-Gate

Imported from [issue #807](https://github.com/Knatte18/loomyard/issues/807).

## What was seen

Reported by the batch 07 recovery strand of run `csharp-alphabet` in the quarry hub (Knatte18/quarry).
Under `lyx gate test`, `runtime.Caller(0)` returns a module-relative path (`github.com/Knatte18/quarry/...`) instead of an absolute one, as a `-trimpath` build would.
Every test that locates its repository through it then fails with "not an absolute path".
It reproduces on `./internal/repopath`, which the batch never touched.
`lyx gate test -vv` shows no `-trimpath` in go's arguments, so the flag probably reaches go through the slot environment.
A raw `go test -count=1 -tags integration` over the same packages passes.
`go env GOFLAGS` in the hub prime is empty.

## Workaround

None yet: the recovery reported the batch OK on the raw run and the lints, and every later batch's gate run is expected to hit the same failure.

## Expected

`lyx gate test` builds as plain `go test` does, so `runtime.Caller` yields absolute file paths, or the gate documents and sets any build-path flag it adds.


## From issue #808

Imported from [issue #808](https://github.com/Knatte18/loomyard/issues/808).

What happened: inside a gate slot, `lyx gate test` over several packages failed every test that derives a source location from `runtime.Caller(0)`. The call returned a module-relative path (`<module>/...`) instead of an absolute one, as a `-trimpath` build does, so the code under test failed with "not an absolute path". The failure reproduced on a package the batch never touched.

Evidence it is environmental: `lyx gate test -vv` showed no `-trimpath` in go's args, `GOFLAGS` was empty in the agent shell, raw `go test` (with and without `-tags integration`) passed on the same packages, and the same gate run in a plain `git worktree` outside the hub passed. The cause therefore sits in the gate slot environment, which the agent strand may not read.

Why abnormal: the gate gave no signal about the cards, so agents fell back to raw `go test` for verification, which bypasses the gate's purpose. The task has since changed its tests to resolve roots from the working directory, but any module using `runtime.Caller` for fixture lookup will hit this again.

What to do: find what makes the slot build behave like `-trimpath` (slot env, go env file, toolchain flags) and remove it, or document it and have `lyx gate test` surface it. Having gate output name the effective trimpath/GOFLAGS state would make this diagnosable.

Provenance: task `csharp-alphabet`; notes 01-csharp-canonical-type.md, 07-resolve-csharp-recovery.md, 07-resolve-csharp.md.

## Operator decision (2026-10-22)

The cause is known: commit 2d3841504 makes the gate add `-trimpath` to GOFLAGS on purpose, for test-cache reuse, so `runtime.Caller` returns module-relative paths.
Keep `-trimpath`; document it and surface it in the gate's output, so a test author meets it before debugging a path.
