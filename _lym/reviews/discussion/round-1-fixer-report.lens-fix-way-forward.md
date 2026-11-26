```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked whether each refusal, test and step the fix added can be carried out from the state it names.

Ways forward checked:
1. D13: a required seam left unset makes the verb refuse, naming the seam.
2. D12: a core row loads config values through `internal/configengine`.
3. D10: before-and-after measure in one session.
4. D5/D13 criterion: a test proves the status file bytes are unchanged.
5. D13 criterion: a `cmd/lym` test shows the builder is installed.
6. D4 criterion: the mutations array is unchanged.
7. D3/D4: inverse deletes present a token.
8. D2: the `holdTeardown` snapshot path.
9. D13 guard: the dependency-closure check.

Items 6 to 9 can be reached and lead somewhere. Findings on 1 to 5 follow.

**MEDIUM — D12's new config sentence: a core row cannot reach fabric's config spec.**
- Changed text: "A core row that needs a value fabric reads from its config … loads fabric's config through `internal/configengine` and hands the loaded value to the snapshot builder; a row that varies such a value writes a config file and loads it the same way."
- `configengine.Load(dirs Dirs, spec Spec)` (`internal/configengine/config.go:172`) needs fabric's `Spec`. That spec is built in `internal/fabricengine/config.go:73`.
- A `fabriccore` test can only get it by importing the engine. An in-package import is a cycle, since the engine imports the core. An external test package would link git and disk into the tier0 core tests, which D1 and D6 forbid.
- No decision moves the spec into `fabriclayout` or `fabriccore`, so a core row cannot be written the way the sentence says.

**MEDIUM — D13's required-seam bullet: fabric's own tests of the shed-codec and stencil-judge verbs are left with no legal way to run.**
- Changed text: "A seam fabric needs to keep today's behaviour is required: the run-records paths, the run locks, shed's codec and the stencil ordering judge. A verb whose run would use one refuses before acting when it is unset."
- The only way past the refusal is to supply the seam.
- A fabric test cannot supply shed's real codec or the real stencil store: D12 bars importing them, and the guard criterion fails on `internal/shedengine`, `internal/shedrun` and `internal/stencilstore`.
- It also cannot hand-write a codec or judge, because D12 bars "fabricated stand-in data for another real module, such as a hand-written shed run state or another engine's output". The workspace builder got the opposite treatment: it was made optional for exactly this reason.
- The record never says where the tier1 tests of the takeover write, the code-tip settle and the warp forward go: into the supplying module, or as action-kind tests that never call the codec. As written, D5's "a test is deleted only when its replacement lands" cannot be met for those verbs.

**MEDIUM — D10's new measure step: measuring "the commit before part 1" needs a second checkout of an old tree, and the record names no way to get one.**
- Changed text: "It measures before and after on one machine in one session: the commit before part 1 and the final tree, the median of at least two tier1 runs each."
- A lym implementer may not create a worktree, clone or checkout unless its parent or the operator says so (`.lym/shuttle/e31f00541d08d0a5e5651cbaea605ad2/prompt.md:47`). Its gate runs only on its own worktree.
- D10 does not say who provides the before-tree checkout or how the close-out part asks for it, so as stated the step stalls at a refusal.

**LOW — new D5/D13 criterion: the byte-identity test names no home package.**
- Changed text: "A test written against today's code in the part that cuts it shows a takeover write and a code-tip settle leave the same status file bytes after the cut."
- Before the cut the test must call fabricengine's takeover write and code-tip settle. After the cut it must supply shed's codec. Fabric's test files may import neither shed nor the codec (D12), so the test can only live in shed's module or `internal/fabriccli`.
- Without a named home, a reader may put it in `fabricengine`, where the D12 guard then fails it.

**LOW — new `cmd/lym` criterion: no way named to observe the install.**
- Changed text: "A test in `cmd/lym` shows the binary installs the workspace builder before it runs a verb."
- The installed hooks are held in unexported atomics with no getter (`internal/fabricengine/hubworkspace.go:86,110`).
- The record says neither "run a verb and check the workspace file is written" nor "an exported read of the installed builder", so the criterion has no stated observation to assert on.

Tool calls:
- Read `internal/configengine/config.go:150-190`, grepped fabricengine for `configengine.Spec{` and for which files hold `secret_paths`: my context did not show how fabric's config is loaded or where its spec is built, which the D12 way forward depends on.
