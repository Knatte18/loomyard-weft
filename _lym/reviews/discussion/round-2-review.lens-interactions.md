```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked pairs of decisions over shared state, then checked each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 × D2 (where the file's directory comes from): broken. D1's claim that the file sits "beside `slack.json`" fails for a relative `XDG_CONFIG_HOME`. See F2.
- D2 × D3 (no directory, or a relative variable, means the hostname or an error): held.
- D3 × D5 (a bad file makes `Name` refuse, and the verb repairs it through `ReadFile`): held.
- D3 × D6 (the name is read on every call, while the file write comes before the sweep): held. D6's window bound covers the gap.
- D3 × Slack constraint (the relay keeps the old name, and `retireEntry` runs under the new name after restart): held.
- D4 × D3, D6 and D7 (the name rule against the `names` entries that feed the replace set and the landing match): broken. See F1.
- D5 collision check × D5 repair branches (which names count as this machine's): broken. See F3.
- D5 collision check × D6 sweep (same hub list, a pull before each): held.
- D5 re-run × D6 (name already set: nothing written, sweep runs again): held.
- D5 chained renames (A→B with a failed hub, then B→C) × D6 replace set: held, because `names` accumulates.
- D5 lost names × D6 and D7 (owners left under a lost name, landings under it not counted): held. D5 states both effects.
- D6 replace set × `Takeover` unset in `writeOwners` (prime and slug paths): held as designed.
- D6 × D7 (both read the same former names from `names`): held. They are consistent with each other.
- D8 × D3 and D5 (path seams, other packages read the real file): held.
- Criteria 199–221 against D1–D8: held, apart from the gaps in F1 and F3.
- Constraints (leaf imports, tolerant records, string equality, Slack) against D2, D3, D6 and D7: held.

**Findings**

F1. MEDIUM · design · D4 × D3/D6/D7 (`decision-record.md:81-83`, `:73-74`, `:129-131`, `:160`)
- **Summary:** former names in `names` are never checked against the name rule, yet D6 uses them as the replace set's authority.
- **Detail:**
  - D4 applies the rule only to "a name set in the file". D3's `machinename.ReadFile` (new) and `machinename.Names` (new) return `names` without validation.
  - D5 itself sends the operator to hand-edit the file: "restoring the lost name in `names` by hand", and hand-writing `{"name": …}`.
  - An empty string or junk entry in `names` therefore reaches D6.
  - Owner `""` is the recorded owner of every beam project without one, and of a hub with no prime owner (`pattern/PATTERN-owner-machine.md:29-30`).
  - D6 selects "every open beam project and the prime whose recorded owner is in the set". The sweep would therefore claim every ownerless beam and the unowned prime for this machine.
  - That breaks D5's bound that the verb "never writes an owner recorded under any other name", and changes ownership those entries inherit from their parent.
  - The same unchecked entries also feed D7's landing match.
- **Fix:** D3/D6 apply `machinename.CheckName` (new) to every former name, and drop or refuse an invalid one.

F2. LOW · consistency · D1 vs D2 (`decision-record.md:29-30`, `:39`, `:45-48`)
- **Summary:** D1 says the file lives in the directory `githubclient.CredentialDir` resolves, beside `slack.json`. D2 resolves it through `standalonestate.AbsoluteUserConfigDir` (new) instead.
- **Detail:**
  - For a relative `XDG_CONFIG_HOME` the two differ: the credential files resolve under the working directory, the machine file under `$HOME/.config/lym`.
  - D1's bound names only the absolute-`XDG_CONFIG_HOME` divergence.
  - Evidence: `cacheDir` in `internal/githubclient/cache.go:48-50` takes any non-empty `XDG_CONFIG_HOME` as given.
- **Fix:** D1 states the relative-variable case, or drops the "beside" claim.

F3. LOW · consistency · D5 repair branches × collision check (`decision-record.md:100`, `:106-112`; criterion `:212`)
- **Summary:** the two repair branches disagree on which names are this machine's, and the collision check's own set over a bad file is never stated.
- **Detail:**
  - Over a file that does not decode, the write records the hostname rule's name as the old name, and D6 sweeps it. The collision check runs before that write, though, and D5 does not say whether it counts the hostname name as this machine's.
  - So a repair back to the hostname name, while boards hold it, reads as refused under D5's text, even though the same run is about to treat that name as a former name.
  - Over a file that decodes with an invalid `name` and an empty or absent `names`, as a hand-made file has, the hostname rule's name is not recorded.
  - In that case owners written under the hostname before the hand file stay unswept, and a repair back to the hostname name is refused.
- **Fix:** D5 names the set the collision check uses over a bad file, and treats the hostname rule's name the same in both repair branches.

**Tool calls:** none.
