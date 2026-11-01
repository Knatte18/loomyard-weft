```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked each pair of decisions that act on the same state, then each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 claim and lock × D3 lease written before the spawn: held. The lease exists before the runner's first read, and a start failure expires it.
- D1 join rule × D3 lease renewal: held. A front that finds a passed lease supersedes, and the runner ends at its next read, within 1 s.
- D1 marked exit file × D3 abandon by lease, supersede or signal: held. A marked file is never replayed, and the next call supersedes.
- D1 delivered mark × D3 front signal: held. A signalled front marks nothing, so a later front can still take the result.
- D1 supersede × D3 slot hold: held. Two runners of one key overlap for up to about 1 s plus kill time, as D3 states, and the pool bound still holds.
- D1 claim wait and lock × D4 cap validation: broken (F1).
- D1 claim wait × D5 long cap: held. 55m plus 30 s plus the lock wait still fits under the 60m Bash ceiling.
- D3 lease monotonicity × D5 short and long caps: held. A later deadline wins, so a short-cap front never moves a long-cap lease earlier.
- D3 lease grace × D5 long cap: held. The bound "long cap plus grace" is stated.
- D5 cap pick × D3 lease write: held. The front picks its cap before it writes the lease, and the order is implied.
- D5 `Abandoned` × D3 cancel causes: held. A lease cancel, a supersede and a signal all cancel `runTest`'s ctx (`internal/gatecli/run.go:496`).
- D2 strand removal × D6 `--wait` gate line: held. The child sees no strand and has `--wait`, so it runs in-process.
- D2 inherited slot × D6 non-gate line: held. A nested gate call in the line inherits the runner's slot and runs in-process.
- D1 join key × D6 card key: broken (F2).
- D1 "with `--wait` semantics" × D6 runner: broken (F3).
- D3 signal handling × D6 card-verify runner: held as a requirement. `runCardVerify` catches no signal today (`internal/gatecli/cardverify.go`), and D3 binds the runner to do so.
- D5 × D6: held as stated. A card-verify call has no `--gate`, so it always keeps the short cap.
- D7 render × Constraint "Go gate calls carry `--wait`": held. `verifyFixGate` feeds only the prompt render (`internal/websterengine/runlevel.go:699,732`).
- D7 × D6 `**Verify:**` keeps `--wait`: held, since `IsGateCall` requires it.
- D4 fallback × D2 non-zero cap: held. The fallback is non-zero, so a call whose config fails to load still detaches, and the D4 AC says so.
- D4 `lym shuttle await` default × D8: held. The template equals today's constant.
- D9 × D1 and D6 answers: held.
- ACs D1 to D9 against their decisions: consistent, except that no AC covers F1's case.
- Constraints against D1, D3 and D6: held.

**Findings**

F1
- severity: MEDIUM
- class: design
- location: `decision-record.md` D4, cap validation, against D1's claim and lock wait
- summary: D4 accepts any role cap below the role's lifetime. D1 lets a front block on the key lock and on the runner's claim, up to 30 s each, before its cap-long `AwaitAny`. A starting front can therefore block for its cap plus 30 s, and a front queued on the lock for its cap plus up to 30 s more. A valid cap such as 290s on a 5-minute lifetime returns cold. The validation should require the cap plus the claim bound to stay below the lifetime, or the claim wait should count inside the cap.
- evidence: `decision-record.md:49` (lock held until the claim, at most 30 s), `decision-record.md:57` (`AwaitAny` for at most the cap), `decision-record.md:111` (cap only "below the lifetime").

F2
- severity: LOW
- class: consistency
- location: `decision-record.md` D6 key against D1 key
- summary: D1 forms the key from "the verb, the resolved run directory and the argument words as given". D6 keys card-verify "by the verb and the card id". `runCardVerify` accepts both `4` and `4-slug` for one card (`cardMatches`), so under D1 the two forms give two runs of one card, and under D6 it is unclear whether the id is normalized. The record should state which key card-verify uses.
- evidence: `decision-record.md:45`, `decision-record.md:145`, `internal/gatecli/cardverify.go` (`cardMatches`, and the help example `4-gate-run-log`).

F3
- severity: LOW
- class: consistency
- location: `decision-record.md` D1 runner verb, against D6
- summary: D1 says the runner runs "the same request in-process with `--wait` semantics, slot wait and tiers included". For card-verify there is no `--wait` flag and no tiers. D6 runs the line through `cardverify.Runner`, which takes a slot without a cap or not at all. D1's description of the runner covers only the `lym gate test` request and should be scoped to it, with D6 owning the card-verify body.
- evidence: `decision-record.md:54`, `decision-record.md:146-149`, `internal/gatecli/cardverify.go` (no `--wait` flag).

**Tool calls**
1. Grepped for `fix_gate` and `FixGate` in `internal/`, to confirm that the rendered `webster-fix` call feeds only the prompt and no Go verify, which D7 against the Go-calls constraint depended on. The manifest did not cover `websterengine`.
