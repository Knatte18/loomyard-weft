```yaml
lens: leanness
outcome: reported
```

Leanness lens review of `decision-record.md` and `support-log.md`: what to move or cut.

1. **MEDIUM — Open risks.** The single open risk restates a choice the writer already made, and it should be cut.
   - Evidence: the `lym tender pause` pair refusal appears three times: Scope "Out" bullet 1, the last Auto-mode assumption, and Open risks.
   - Open risks is for questions only the run's parent can settle. Keeping the refusal is the writer's own pick, already recorded with its rationale in Scope and as Q11 in the ledger.
   - Fix: drop the Open risks bullet, or replace it with "None". Keep it once in Scope Out.
   - Nothing the plan writer needs is lost, so the completeness test passes.

2. **LOW — Scope "In".** Each "In" bullet re-summarises a decision almost word for word: detached worker, suppression, adopt pause, owner before deploy, `--no-orch`, status, `--here`, archive.
   - Evidence: compare the Scope bullets with the opening sentences of D1, D4, D5, D6, D7, D9, D10 and D11/D12.
   - Fix: shrink "In" to one line per area with its decision id, or to a single line saying the scope is D1–D12. Keep "Out" as it is.

3. **LOW — Constraints bullets 3–4 and 6–7.** They restate PATTERN entries that D1, D2 and D11 already apply in their own text.
   - Evidence: D1 already says the worker is logged, reaped, registered under `runproc.HubOwner` and spawned through `selfexec.LymBinary`, and that the follow follows `PATTERN-daemon-wakeups`. D2 already says "internal/state, tolerant of unknown keys". The refusal and way-forward bullet is a standing repo rule.
   - Fix: cut those four bullets. Keep bullet 1, which is load-bearing because D2's resume rests on it. Keep bullet 2, the machine-local note, and bullet 5, which tells the plan writer that the archive and the archive read are fabric calls.

4. **LOW — D3, last paragraph.** "The envelope reports each run as D5 names, and the operator resumes runs with `lym tender resume`" repeats D5.
   - Fix: cut it.

5. **LOW — Auto-mode assumptions bullets 1, 4 and 5.** They duplicate decision text.
   - Bullet 1, the archive tags, repeats D11's rationale and the support log's rejected alternative.
   - Bullet 4, the live-pair run, repeats a sentence of D5.
   - Bullet 5 is the triple from finding 1.
   - Fix: keep each auto-pick in one place. D11's rationale can carry "tags, not branches" in one clause, and the assumption line can shrink to naming the board-text deviation.

6. **LOW — Notes for the plan writer, bullet 2.** "The legacy `HoldCaller` constant stays for reading old records" is already in D5 ("A live `pauserecord.HoldCaller` entry an older adopt left reads the same way").
   - Fix: cut it.

7. **LOW — D11 and D12 rationale, emphasis only.** D11's rationale sentence "the adopting machine cannot see the old owner's unpushed local commits" and D12's rationale "the new owner sees what the old owner had not pushed" say the same thing.
   - Fix: keep it once, in D11.

Not a leanness finding, noted for the other lenses: D11 keeps a foreign-owned dobby alive while its archive fails. On the old owner no adopt record stands, so D4 does not apply there. That is consistent, but the fork owning bounds should check that D11's "stays up" does not clash with the `PATTERN-owner-machine` rule that a foreign-owned dobby exits.

Tool calls made: none. My context already held both reviewed files and the rubric.
