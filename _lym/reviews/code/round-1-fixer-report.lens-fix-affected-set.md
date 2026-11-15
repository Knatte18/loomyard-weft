```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind the findings F1, F3–F10 and check the fix covers every member.

No findings.

- **F1** (every `WorktreeSeam` implementer and `ArbiterAttempt` writer): `WorktreeSeam{` appears only in `internal/shedarbiter/call_test.go` and `internal/loomcli/arbiter.go`. Both now set `Changes`. The one `ArbiterAttempt` that carries a snapshot is built in `call.go`, and it now sets `Changes` too. The batten arbiter sets no `Worktree`, so it needs no `Changes`.
- **F1** (every place that claims tracked-only or untracked behaviour): a grep for "tracked uncommitted" and "uncommitted change" found only `edits.go`, whose comments were updated, and the fix fragment, which now says "a new file included". No stale claim remains.
- **F10**: `# Arbiter` is the only arbiter section in `internal/loomcli/doc.go`, and it carries the new sentences.

Unchecked: F3–F9. They are test rows, one `//testtiming:keep` marker and a stencil bullet, not sets I could search. I also did not verify that the arbiter's attempt directory sits outside the code checkout's untracked set. `ArbiterDir` resolves under `RunDir`, in the `.lym` tree per `ArbiterRel`, and `.lym/` is gitignored.

Tool calls:
1. Grep for the tracked and uncommitted wording across `internal`, `docs` and `contracts`, plus `Snapshot` in the `shedarbiter`, `battencli` and `loomengine` packages. The context held none of the other doc claims or non-test uses.
2. Read `shedrun.ArbiterDir`. The context did not show where the attempt directory lives.
