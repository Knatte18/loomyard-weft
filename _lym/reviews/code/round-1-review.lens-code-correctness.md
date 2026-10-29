```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the diff, `internal/websterengine/recoverbatch_test.go` and `recoverbatch_helper_test.go`.

**List: function or branch, case run, verdict**

- `uncheckableRefusal` setup, case `atStartSetup` row: the setup builds a local `rec`, writes it to `fx.Deps.State.Batches[1]` and `fx.Seeded`, and returns it. `atStartSetup` then sets `.StartSHA` on that same pointer. Correct. No closure variable is written any more, so parallel rows cannot cross-write.
- `uncheckableRefusal` check, `atStartBase` run from the at-start row: `rec := fx.Seeded` is the pointer this row's own setup stored. The identity test `Batches[1] != rec` compares the pointer the code under test left in place with the seeded one, as before. The Terminal, Status and StrandGUID assertions are unchanged. Correct.
- `committedBaseSetup, _` (check discarded), `committedSetup` rows (two): `fx.Seeded` is set on the row's own fixture and nothing reads it, because those checks never call a base check. Correct, with no stale-read path.
- `longBesidePathSetup` and `longBesidePathCheck`: the inner `setup(fx)` sets `fx.Seeded` on the row's fixture, and `longBesidePathBase` reads it from `fx` in the same subtest. Correct.
  - `longBesidePathBase` and `scratchPath` are still closure variables written in a setup. Only one row uses `longBesidePathSetup`, so there is a single writer and no race. The record scopes these out.
- `noStartCheck(base)` wrappers (`fabricRefusalCheck`, `noStartCheck(longAloneBase)`) and `pauseRefusalCheck`: each runs the base check with its own `fx`, which its paired setup seeded. Correct.
- Nil boundary: a check that reads `fx.Seeded` without its setup having run would dereference nil, since `rec.Terminal` follows `rec := fx.Seeded`. I walked every refusal check's row and each is paired with its setup, so no row hits this. This is the one remaining way the change could fail; it is not a defect today.
- Fixture lifetime: the subtest loop calls `newRecoverFixture(t)` inside each `t.Run` (shown for the three loops I could see at lines 188-191, 591-594 and 1724-1727), and the helper returns a new struct each call (`recoverbatch_helper_test.go:130`). So `Seeded` is per-subtest. I could not see the `TestRecoverSpawnOrAttach` loop itself, which is around line 1600-1700.
- `Seeded` field in the helper: the new nil-initialized field is not set by the one `recoverFixture{...}` literal at `recoverbatch_helper_test.go:130`, so other tests building a fixture are unaffected. Correct.

No findings.

**Tool calls (2)**
1. `awk` over lines 1600-1700 of `recoverbatch_test.go`, plus `grep "range cases" -A14`: to confirm the fixture is built inside each subtest. The awk matched nothing and the grep showed only three other loops, so the `TestRecoverSpawnOrAttach` loop stays unconfirmed. Earlier context did not show where it builds its fixture.
2. None other.
