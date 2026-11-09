```yaml
lens: fix-proposal
outcome: reported
```

Task: as the fix-proposal lens, challenge each review proposal the fixer carried out as proposed.

Proposals checked: F1, F2, F3, F4, F5, F6, F7, F9, F10, F11, F12. F8 was fixed with one of the review's two options.

**1. MEDIUM. F2, D5: always counting the hostname rule's name as this machine's makes one machine sweep another machine's owners when two machines share a short hostname.**
- Changed text: D5, "It takes the hostname rule's name, `machinename.FromHostname`, as one of this machine's names beside the current and former ones", and "the old name and the hostname rule's name are each appended to `names` when absent from it". Also the new criterion "over a hand-written file holding only the name `osl46` records the hostname rule's name in `names` and sweeps owners recorded under it".
- The case: `FromHostname` keeps only the part before the first dot (`internal/machinename/machinename.go:23`), so hosts `osl-1046.a.example` and `osl-1046.b.example` both read `osl-1046`. Telling them apart is a main reason to name a machine.
  - Machine A gets its file by hand as `{"name":"a46"}`, then runs `lym machine name a46`.
  - `osl-1046` now goes into A's `names` and into D6's replace set, so A's sweep moves every open entry of machine B onto A.
  - The collision check counts `osl-1046` as A's, so it cannot refuse.
- What got worse: before the fix, writing the file by hand was a way to take a name without sweeping the shared hostname name. Now no path avoids the sweep, and every re-run appends the name again.
- The bound does not cover it. D5's shared-name bound covers "a former name another host has since taken as its own, by hostname or file". Here the name is shared from the start, not taken later, and the fix never added that case.
- Evidence: D5 Bound text; `machinename.go:23`.

**2. LOW. F1, D3 and D5: the exemption is tied to the current hostname, so a changed hostname drops a former name owners still hold.**
- Changed text: D3, "It drops every entry of `names` that is empty or fails `machinename.CheckName` (new), unless it is the hostname rule's name". D5, "every entry that is empty or fails D4, other than the hostname rule's name, is dropped".
- The case: the host was `osl_1046`. `FromHostname` lowercases and cuts at the dot but keeps `_` (`machinename.go:23-24`), and D4 does not check its output.
  - A first rename records `names: [osl_1046, osl46]`, and one hub's sweep fails.
  - Later the hostname changes to `osl-1046`. `osl_1046` now fails D4 and is no longer the hostname rule's name.
  - `machinename.Names` drops it, and the next run of the verb erases it from the file.
  - The failed hub's owners stay stranded under `osl_1046`, and D7 stops counting landings made under it.
- Only the empty entry causes the harm F1 describes. A non-empty invalid entry matches only an owner recorded under exactly that string, and only this machine could have recorded one, so dropping it buys no safety.
- Evidence: F1's own issue text, where the harm cited is the empty owner read as no owner (`internal/boardengine/owners.go:175-177`).

I made no further tool calls.
