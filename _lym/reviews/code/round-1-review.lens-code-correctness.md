```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I ran each function and branch the diff changes against concrete inputs.

**Per function or branch: the case I ran, and whether it is correct**

- `boardengine.ValidateEntrySlug` (`internal/boardengine/task.go:409`):
  - a 32-character slug → nil;
  - a 33-character slug → the length error, same text as before;
  - `"root"` → the reserved error;
  - a 33-character slug → the length error comes first, the order the old code used;
  - `""` → nil, which is correct because `NewTask` refuses an empty slug before it calls this.

  Correct.
- `boardengine.NewTask` (`task.go:430`): the empty, length and root cases behave as they did at the base. The error is returned unwrapped, as before. Every `NewTask` caller (`store.go:602`, `:994`, `:1175`) sees the same behavior. Correct.
- `raddleBoardContext` new `fabricengine.LoadConfig` step (`internal/loomcli/raddlecontext.go:61`):
  - On success, `fabricConfig.Dirs()` is captured for the closure.
  - On failure, it returns `BoardContext{}` and the error, unwrapped like the `boardengine.LoadConfig` error beside it.
  - The board config load just before it, on the same dirs, already proved the hub is initialized, so the only new failure is a corrupt `fabric.yaml`. That is a real error and is reported.

  Correct.
- `ValidateSlug` closure (`raddlecontext.go:85`):
  - `"part-a"` → nil;
  - a 33-character slug → the length error;
  - `"root"` → the reserved error;
  - `"part/a"` → the pair rule's single-path-component error;
  - `""` → `ValidateEntrySlug` admits it, then the pair rule refuses it as empty;
  - a slug ending in the weft suffix → the pair rule refuses it;
  - a junction name → the pair rule refuses it as reserved.

  Correct.
- `checker.slugs` new `slug-invalid` branch (`internal/raddleparser/check.go:506`):
  - nil hook → skipped, no panic;
  - a hook refusing a part slug → one `slug-invalid` finding with `Part` = the slug and the detail `"the board refuses the slug: &lt;err&gt;; way forward: …"`;
  - a refused project slug (first in `filedSlugs`) → reported;
  - a perch project whose slug is the run's own entry → that entry is already on the board with a pair, so the hook admits it and nothing is reported. Correct.
  - a refused slug filed twice → one `slug-duplicate` and two `slug-invalid` findings for the same slug. This is the same doubling `slug-taken` already has (`check.go:500`). See the NIT below.
- Raddle-File in-write re-check (`internal/loomshed/raddlefile.go:481`): it builds `BoardContext` without `ValidateSlug`, and `RecheckBoard` never reads the hook. Behavior is unchanged, no nil dereference. Correct.
- `envkit` stub context (`internal/testkit/envkit/envkit.go:372`): the zero `BoardContext` has a nil hook, so `slugs()` checks nothing. Correct.
- Test fixture hook (`internal/raddleparser/fixture_test.go:166`): refuses only `refusedSlug`, and every other row's slugs pass. Correct.

**Findings**

1. NIT, `design`, `internal/raddleparser/check.go:506-509`. A refused slug filed twice gets two identical `slug-invalid` findings, one per occurrence, beside its `slug-duplicate` finding. It is cosmetic and matches how the existing `slug-taken` branch at `check.go:500` already doubles. The check name and `Part` are the same in both, and the author fixes it with one rename. Evidence: the loop over `filedSlugs` at `check.go:494` runs the hook for every occurrence, and `seen` gates only the duplicate finding.

No other correctness defect found. No error is swallowed or misreported, and no failure path leaves state inconsistent: the context builder writes nothing, and `Check` is pure.

**Tool calls**

1. `grep` of `internal/raddleparser` for empty-slug handling: to see whether an empty slug would be reported twice once the pair rule refuses `""`. My context had no check for empty slugs, and the grep found none, so the new branch is the only report for an empty slug.
