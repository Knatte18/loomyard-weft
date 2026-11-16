```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked the changed text against its sibling decisions, criteria and cross-references.

**1. LOW: the parent's Added-after-Discussion entry still says "pushed state".**
- Changed text: the D9 bound now reads "D1 leaves every transition pushed or owed", and the D9 rationale reads "at a transition pushed or owed".
- Issue: "Added after Discussion (parent, 2026-11-16): The hub stop writes no per-run pause or cancel intent" still says "its guarantee that every run is left in a pushed state rests on D1's synchronous pushes".
- Evidence: the F10 change was made in D9 but not in this twin. The entry is the parent's text, so the report should name the mismatch rather than edit it.

**2. LOW: the D5 criterion still promises the owed push lands.**
- Changed text: the new D5 bullet says "Nothing orders that push before an adopter's takeover", and the new D2 bound line says "An owed push that reaches origin after the takeover is rejected and archived".
- Issue: the D5 criterion "under a hold, a run whose push is owed on a machine other than the adopter's gets no `bolt-hold` entry, its owed push lands once origin answers" was left as it was. After the F7 change, that push can be rejected and archived instead of landing.

**3. LOW: the D2 and D4 hand-over marks are written at different points.**
- Changed text: D4 step 4 now "writes D2's hand-over mark once its archive lands", before the local removal.
- Issue: D2's in-loop hand-over still writes the mark "once the steps above finish", and those steps include the local removal. So the gap F11 raised (a removal that fails leaves no mark) stays open for D2's own path. D2's twin was not moved to "once its archive lands".

**4. LOW: the new registration triggers clash with "only the owner setter writes the map".**
- Changed text: D3 now says "A machine registers at its first catch-up (D4) or dobby start after the upgrade, at its first owner write and at `lym machine name`".
- Issue 1: D3 still says "Only `boardengine`'s owner setter writes the map". A registration at a catch-up or dobby start writes no owner, so either that writer or the sentence has to change.
- Issue 2: D4 step 1 reads the registry but does not say the catch-up registers.

**5. LOW: a D3 criterion assumes `--forget-former` always passes.**
- Changed text: the new `--forget-former` refusal, which applies while an owner, takeover entry or unfinished cancel intent still names the forgotten name.
- Issue: the criterion "after that verb, the registration lands" was left as it was, and the verb can now refuse. The criterion could state the case with no records under the name, or have the refusal cleared first.

Further tool calls: none.
