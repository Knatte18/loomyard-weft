```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: I checked whether each way forward the fix introduced can be carried out from the state it names. I found nothing to report.

- **`lyx beam start <slug>` after a failed dobby restart (D9, F2).** It can be reached. The failed-re-check path writes the run status naming Beam-Await before it tries the restart, so the landing hold is lifted. D3's only hold refusal is "a beam whose pawl run holds the landing hold", and D3 reads no dobby-stop marker. So the step passes and starts the dobby. The marker it leaves standing does no harm: the next accept writes it again anyway, and a passing re-check removes it.
- **"The next accepted request restarts the dobby on its own failed re-check".** It can be reached. A new accept rewrites the marker and stops the dobby (which is already down). A failed re-check then restarts it because the marker stands, and a passing one goes on to land with the dobby stopped, which is what the landing wants.
- **The driver-dead notice naming `lyx loom start` once the cap has passed (D9, F1).** It is reached only when the cap's respawns pass with no status-file write between them. Below the cap the notice carries the plain "driver strand is dead" text, and the amended criterion asserts only that one notice is sent per episode. So the fix promises no remedy that is never sent.
- **New acceptance criterion for a failed restart.** It describes a state D9 now produces: the marker stands and the orch notice names the blockers and `lyx beam start <slug>`.

No findings.

Calls made:
- grep of D3 and of the Go tree for dobby-stop marker handling: my context did not show D3's refusal list, or whether any code or decision reads the marker and would block `lyx beam start`.
