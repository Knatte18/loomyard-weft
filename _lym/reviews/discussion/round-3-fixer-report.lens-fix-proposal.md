```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged F37's proposal (stop identity, the count by identity, and the `halt:` key built on identity), which the fixer carried out as proposed.

**MEDIUM: a driver that dies before writing its stop record takes over the previous stop's identity**
- **Changed text:** D3: "A restarted driver that reads a stopped run whose stop key matches the record's open stop takes over that identity, so a driver restart opens none."
- **Case:**
  1. A launch fails at row R. The run is `failed` with stop key K = (R, n, failed), and the driver opens identity 1.
  2. The first failure is handed back and the orch resumes. The run runs again under the same history length n, because a producer error appends no entry.
  3. The step fails again, and the shed persists `failed` with stop key K. The driver dies before it writes identity 2 to the stop record.
  4. The respawned driver reads a stopped run whose key K matches the record's open stop. It takes over identity 1.
- **Result:**
  - The second failure counts as the first, so the third halt at the row arrives one stop late.
  - The halt carries the same `halt:` key, so D8's dedup and D7's posted record mute it, which is the defect F37 set out to remove.
  - D3's criterion "a driver restarted at a stopped run keeps the stop's identity" pins the wrong behaviour for this case.
- **Evidence:**
  - The two failures share K because `StopKeyOf` is (producer, `len(History)`, state) (`internal/sheddriver/stopkey.go:14`), and `appendHistory` appends nothing on error (`internal/shedengine/run.go:340-357`).
  - The changed text says when an identity opens. It never says what closes the record's open stop, such as a resume, a re-step or a fresh start that runs the row again.
- **What would close it:** the stop record closes its open stop when the driver steps the row again. Only a record whose open stop the driver never stepped past may be taken over.

No other proposal broke under the cases I built:
- The third-failure walk (fresh start, hand-back, resume, failure) reaches count 3 and `settle: operator`.
- Folds and the excluded waits count as D3 states.
- A `halt:` key of row plus identity is distinct per stop for D8 and D10.

Further tool calls: none.
