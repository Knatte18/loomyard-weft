```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix's change set for consistency with its surroundings (fix-consistency lens).

Findings:

- **LOW**: `contracts/stencils/stencils.go`, the `Registry` doc comment.
  - It now reads "Init registers this registry with stencilstore". "Init" is capitalised mid-sentence as if it were a name, and the identifier is lowercase `init`. It also describes `init`, which the `init` doc comment below it already covers.
  - Evidence: the edited `Registry` comment against the `init` comment two lines below.

- **LOW**: `internal/stencilstore/doc.go`, line 12, still says "the caller's Older ordering reports the recorded writer older".
  - The new paragraph at lines 32–33 introduces a second ordering hook, `RegisterOrdering`, for the read path. It never says that `RegisterOrdering` takes the same function as `Source.Older`. A reader of the package doc cannot tell that the read path and the refresh share one ordering.
  - Evidence: `doc.go:12` against `doc.go:33`, and `cmd/lym/stencilseed.go`, which passes `StencilSource(...).Older`.

No stale references to the removed symbols remain.
- A repo-wide grep found no `contentForRunning`, `shippedOverNewer` or `newerCopyWarned`.
- The `internal/shedadapters/bouncer.go:330` comment says "stencilstore.Read never falls back to a shipped default". It is true again now.
- The `Read` and `readInclude` call sites both use the new refusal. The `Read` doc, `doc.go` and the `skew.go` header all describe a refusal and agree with each other.

Calls made (2 of 6):
1. Grep for the removed symbols, fallback wording and `Older`/`RegisterOrdering` references. My context had no repo-wide view of stale mentions.
2. Grep for the stencil-pins notice, plus `git show --stat HEAD`. I needed to confirm the commit's file list and where the notice text lives.
