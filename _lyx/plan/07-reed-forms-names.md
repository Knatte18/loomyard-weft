# Card 7 — reed-forms-names

**Edit:**
- `internal/reedengine/strand.go`
- `internal/reedengine/replace.go`
- `internal/reedengine/name.go`
- `internal/reedengine/byname.go`
- `internal/reedengine/config.go`
- `internal/reedengine/geometry.go`
- `internal/reedengine/template_posix.yaml`
- `internal/reedengine/template_windows.yaml`
- `internal/reedcli/add.go`
- `internal/reedcli/remove.go`
- `internal/shuttleengine/run.go`
- `internal/shuttleengine/spec.go`
- `internal/reedengine/strand_test.go`
- `internal/reedengine/name_test.go`
- `internal/reedengine/byname_test.go`
- `internal/reedengine/config_test.go`
- `internal/reedengine/lock_test.go`
- `internal/reedengine/spawn_test.go`
- `internal/reedengine/launchscript_cleanup_test.go`
- `internal/reedengine/ensuresession_integration_test.go`
- `internal/reedengine/mouse_boot_integration_test.go`
- `internal/reedengine/contract_integration_test.go`
- `internal/shuttleengine/run_test.go`

**Prosa:**
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/agentname/agentname.go`

**Intent:**
reed forms every strand's full name inside `AddStrand`, under its state lock, from the prefix its geometry tells it, and stores it in `Strand.Name` as a birth attribute.
The `strand_name` template and its `<ROLE>`, `<ROUND>` and `<SHORT_GUID>` tokens retire.

Up-front checks, before the session pre-flight so a refused call never boots a tmux server, in both `AddStrand` and `ReplaceStrand`:
- an empty `Geometry.NameCode` refuses with `no strand name can be formed: this hub records no short code; way forward: lyx fabric code <code> records it, then retry`, and a recorded code failing `agentname.ValidateCode` refuses naming it with the same way forward;
- a `NameSlug` failing `agentname.ValidateSlug` refuses naming the worktree and the grammar, `way forward: lyx fabric add <slug> creates the task under a slug that fits`.

Formation (`strandNameLocked`, called from `addStrandLocked`; `resolveStrandName` and `FormatStrandName` are deleted, `newGUID` stays in `name.go`):
- `AddSpec.NameOverride` is an explicit name, a role segment or a full name, read through `agentname.Resolve` against the told code and slug.
  When a strand in this worktree's state already holds it, the add refuses naming the holder's guid, `way forward: lyx reed remove --name <role> frees it, or pass another role`; it is never renumbered, since an explicit name is one a process means to address.
- Otherwise `AddSpec.Role` is numbered with `agentname.NumberRole` against the role segments of every strand in state, live or dormant (a legacy name `agentname.Parse` rejects holds nothing).
- A spec naming neither takes reed's own default role `strand`, so `lyx reed add --cmd …` keeps working for an operator.
- `agentname.Format` joins code, slug and role.
`AddSpec.Round` is removed, since the round leaves the name; `shuttleengine` stops passing it (`Spec.Round` stays, for its run directory), and `Spec.NameOverride`'s doc says it is an explicit role or full name.

`--if-absent` matches on the full name `agentname.Resolve` gives for `NameOverride`, so a role-segment `--name` and a full one hit the same strand; `validateIfAbsent`'s comment drops the `<SHORT_GUID>` rationale for the real one: an auto-numbered role never matches the strand it would duplicate.
`guidByName` matches with `agentname.Matches`, so `reed remove --name` takes the role segment or the full name, and a legacy exact name still resolves.

`Config.StrandName` and both templates' `strand_name` line are removed; `LoadConfig`'s non-strict decode ignores the key in an older `reed.yaml`, and `lyx config reconcile` drops it as a key the template no longer has.
`reedcli add` loses `--round`; its help and `--role`/`--name` flag texts describe the name grammar, `-N` numbering, and the explicit-name refusal; `remove`'s `--name` help says it takes a role segment or a full name.
`Geometry.WorktreeRoot`'s doc stops naming the `<WORKTREE>` token.

`refusal-spec.md` gains two correctness-halt rows, in the shed section (which loom inherits) and in the webster section, since every producer and Master spawn reaches them: `strand spawn: no short code`, way forward `lyx fabric code <code>`, and `strand spawn: worktree name not a slug`, way forward `lyx fabric add <slug>`.

Fixtures that build a hub-mode `Geometry` and reach `AddStrand` (`newTestEngine`, `newColdScratchEngine`, `newIntegrationEngine`, the contract test's literal) set a test code and slug.
Tests in `strand_test.go`, all against the fake tmux: the first strand of a role is `<code>:<slug>:<role>`, the second `<role>-2`, a dormant strand still counts, an empty slug gives `<code>:<role>`;
an explicit name already held refuses and a foreign-prefix full name refuses;
a missing code and a bad slug each refuse, with the exact way-forward text, and issue no tmux command;
`--if-absent` with a role segment matches the strand a full-name add created.
`byname_test.go` covers role-segment, full-name and legacy-name lookups; `config_test.go` loads a `reed.yaml` still carrying `strand_name`.

**ImpactSummary:** Changes how every reed strand is named and what AddSpec carries; reedcli add/remove, shuttle's AddStrand call and reed's test fixtures move with it, and name lookups in other packages are retargeted in the next card.

**Commit:** 7: reed: form each strand's full name from the told prefix
