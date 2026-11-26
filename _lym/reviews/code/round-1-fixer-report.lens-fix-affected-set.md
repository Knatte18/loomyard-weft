```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that each set-premise finding's fix covers every member (F1 messages and help, F2 stale wording, F4 reconcile-before-load sites).

**Left unchecked:**
- F1's Go comments and package docs: the parent deferred these.
- F3 and F5–F13: their searches were not rerun, because their sites are single or covered by the build and tests.

**Findings**

1. **MEDIUM** — F1 change set (commit 5c41ffaa4, "name config keys in full in messages, help"). Messages and help text still name keys by file, without the module path. These are not comments, so the deferral does not cover them:
   - `internal/stencilstore/engineblock.go:154`: "set the engine key of shuttle.yaml", which should be `shuttle.engine`.
   - `internal/shuttlecli/run.go:85` (help) and `:204` (flag help): "shuttle.yaml's engine".
   - `internal/boardengine/labels.go:26,43,45,47`, `internal/boardengine/intake.go:253`, `internal/boardengine/board.go:441`, `internal/boardcli/cli.go:333,911`: "the types / labels list of board.yaml", which should be `board.types` and `board.labels`.
   - `internal/gatecli/redfirst.go:117`: "a key of gate.yaml's gates", which should be `gate.gates`. The same "gate.yaml key" wording is in the flag help at `redfirst.go:101` and `internal/gatecli/cli.go:121`, and in the refusals at `internal/shedrecipe/entries_simple.go:40` and `entries_gate.go:103`.

   The cause is that the fixer's sweep pattern took only underscored key names, so single-word keys were missed. The search above is limited to a few phrasings, so there may be more.

2. **LOW** — F2: two comments still say that only a start verb runs the hub walk.
   - `internal/hubreconcile/walk.go:92`: "so the next start verb reseeds a default deleted after a walk".
   - `internal/fabriccli/clone.go:152`: "the next start verb's walk seeds the defaults".

   The reviewer's F2 fix did not list either, but they belong to the same set.

F4: I found no gap. Every `hubgeom.ReconcileBeforeLoad` caller outside the already-tested loom, shed and beam-land paths now has a test. `TestWireDobby…` tests `wireDobby` itself, so it would not fail if `runDobby` stopped passing `wireDobby`.

**Tool calls**
- grep for the "start verb" and "only … removes" wording (F2): my context held only the listed sites.
- grep for `ReconcileBeforeLoad`, `EnsureOrWarn` and the config load calls (F4): the full list of callers was not in context.
- grep of message strings for single-word and dotted key names (F1): the fixer searched only underscored names.
- read of `internal/configengine/doc.go:72-77` plus a grep for keys named by file: neither was in context.
- grep for "the X key/list of Y.yaml" phrasings (F1): not in context.
