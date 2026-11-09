```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens. I checked each pair of decisions that act on the same state, then each criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 × D2 (directory rule, resolver): held.
- D1 × D3 (file location, per-call read, environment bound): held.
- D2 × D8 (the `standalonestate` import, the test seam): held.
- D3 × D4 (the reader applies the name rule): held.
- D3 × D5 (D3 makes `machinename` the only reader of the file; D5 has the verb read and write it itself, through `internal/state`): **broken**, finding 2.
- D3 × D5 (the repair D3 names as the way forward, against D5's collision check): **broken**, finding 1.
- D5 × D6 (file write, then sweep; a re-run sweeps again): held.
- D5 collision check × D6 replace set (local Bolt read before the pull; the replace set limits the write): held, and the bound is stated.
- D5 dropping the new name from `former_names` × D6/D7 (a rename A→B→A): held.
- D5's "former name stays this machine's" bound × D6 sweep and D7 landings (another host holds a former name): held, and the bound is stated.
- D6 replace set × `Takeover` unset × the re-read in `SetOwners`: held (`ownerset.go:142-159`).
- D6 × D3's window bound (per-check readers pick up the new name at their next check): held.
- D6 × Constraint `PATTERN-owner-machine` (owners written only through the setter): held.
- D6 × the criterion "every beam owner … reads the new name": **broken**, finding 4.
- D7 × D5 (former names lost when the file does not decode): **broken**, finding 5.
- D5 × D8 (verb tests against test isolation): **broken**, finding 3.
- Constraint `PATTERN-leaf-packages` × D5's write through `internal/state`: **broken**, part of finding 2.
- Acceptance coverage: D1 to D8 each appear as a whole word in a bullet: held.

**Findings**

1. **BLOCKING · design · D3 × D5 (`decision-record.md` D3 third bullet, D5 collision bullet and repair bullet)**
   The repair D3 names, `lym machine name &lt;name&gt;`, is refused by D5's own collision check in the most common repair: restoring the name the machine already had.
   Take a machine with the valid name `osl46` whose board owners read `osl46`, after its file is hand-broken (`"name": "OSL46"`, or JSON that does not decode).
   - Decodes: D5 drops the invalid name without recording it, so `former_names` lacks `osl46` and no current name reads. `osl46` is then "a board owner that is not this machine's current or former name", and the verb refuses it.
   - Does not decode: the old name recorded is the hostname rule's `osl-1046`, and the same refusal follows.
   - Removing the file, D3's other way forward, gives current `osl-1046` and no former names, so `lym machine name osl46` is refused again.

   Every path leaves the operator on `lym adopt`, the heavy move the record rejects for a rename.
   Evidence: `decision-record.md:54`, `:83`, `:89-92`.

2. **MEDIUM · consistency · D3 × D5 × Constraints `PATTERN-leaf-packages` (`decision-record.md:63`, `:84`, `:89`, `:137`; Notes `:186`)**
   D3 says `internal/machinename` "becomes the only reader of the machine file".
   D5 has the verb read the file "itself instead of through `machinename.Name`" and write it "through `internal/state`".
   The leaf constraint limits `internal/machinename` to the standard library and `internal/standalonestate`, so the `internal/state` writer cannot live there; `internal/machinecli` must then name the file's path and read it.
   That breaks D3's sole-reader claim, and the plan-writer note would pin it as a scan that `machinecli` fails.
   D7 needs the former names from the same file, and no Decision gives `machinename` an accessor for them.
   The record should say which package owns the raw read, the path and the former-names accessor.

3. **MEDIUM · design · D5 × D8 (`decision-record.md:128-131`, criteria `:162-169`)**
   D8 isolates only `internal/machinename`'s own tests and says no test environment variable changes.
   D5's criteria require tests of the verb that write `machine.json`, such as "writes the name `osl46` with `osl-1046` in `former_names`".
   With no seam for the verb and no environment change, those tests write the developer's real per-user `machine.json` and rename their real machine.
   D8 needs to cover the verb's own path seam too.

4. **LOW · consistency · D6 × acceptance criterion (`decision-record.md:103`, `:167`; `internal/boardengine/ownerset.go:188`)**
   `SetOwners`/`writeOwners` skip a done entry, so a done beam project recorded under the old name keeps it.
   The criterion says "every beam owner … equal to the old name on each swept hub reads the new name", with no exception for done beams.
   Either the criterion names open beam projects, or D6 says done ones are left as they are.

5. **LOW · consistency · D5 × D7 (`decision-record.md:91-92`, `:124`)**
   When the file does not decode, D5 loses the former names and bounds only the owners left under them (`lym adopt`).
   D7 then no longer counts landings finished under those lost names, and the record states no bound for them.
   Name the landing effect next to the owner one.

Tool calls: none; the inherited context held every file this lens needed.
