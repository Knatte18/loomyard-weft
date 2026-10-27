# Card 5 — selfexec-callbacks

**Edit:**
- `internal/selfexec#Watcher.Check`
- `internal/selfexec#TestWatcher_Check`

**Create:**
- `plan:internal/selfexec#Watcher.CheckWith` -> `func (w *Watcher) CheckWith(beforeExec func(), failed func(error))`

**Extends:**
- `internal/selfexec/doc.go`

**Intent:**
Decision 3: the dobby's pre-exec line and its failed-exec line and notice need `selfexec` to tell its caller when an exec is about to happen and when one failed.

`CheckWith` is `Check` with two callbacks: `beforeExec` runs right before the exec, after the two-sighting rule and the failed-identity rule have passed, and `failed` runs with the exec's error after a failed exec, once the identity is marked failed.
A nil callback is skipped.
A failed identity is never exec'd again, so `failed` runs at most once per identity, which bounds the dobby's failed-exec notice to one per binary.
`Check` becomes `CheckWith(nil, nil)`, so every other caller is unchanged.
The package keeps its closed import set (PATTERN-leaf-packages): nothing new is imported.

`TestWatcher_Check` gains a column counting `beforeExec` and `failed` calls, run through `CheckWith` on the exec rows: the row that execs once calls `beforeExec` once and `failed` never; the failed-exec row calls `beforeExec` once and `failed` once with the exec's error; the later-different-identity row calls each twice; an unchanged identity calls neither.

`doc.go` gains a sentence naming `CheckWith` and when each callback runs.

**ImpactSummary:** `Check` keeps its behavior by delegating; the new method adds callbacks only, used by the dobby wiring alone.

**Commit:** 5: Tell a selfexec caller before an exec and after a failed one
