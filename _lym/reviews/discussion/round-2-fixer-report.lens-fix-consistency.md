```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix to the D13 seams and shed writes against the sibling text, criteria and cross-references it touches or depends on.

Findings:

1. **MEDIUM.** The new D13 installer paragraph conflicts with Scope Out, which nothing updated.
   - Changed text: "A test outside fabric that reads the hub workspace file, such as those in `internal/beamcli`, `internal/ideengine` and `internal/machinecli`, installs the IDE module's builder itself", and "`internal/hubforge` … calls it when it builds a hub".
   - Evidence: Scope Out still excludes "Other packages' own tests that build hubs through `internal/hubforge`, beyond the import retarget and the tests D12 moves in", and "`internal/hubforge`'s template machinery and its import set". Scope In does not list the edits to the beamcli, machinecli and ideengine tests, or a code change in hubforge.
   - The import set is unchanged, but the record now asks for both changes while Scope forbids or omits them.

2. **LOW.** D9 does not say which part lands the new installer, the hubforge call or the IDE-builder installs in the outside tests.
   - Evidence: D9 says "Each concern part also makes … the D13 import cuts of its own concern's files". The shed cut spans parts 4 and 5 (code-tip settle, takeover), the stencil judge is part 5's warp forward and the workspace builder is part 6.
   - The installer must exist, and hubforge must call it, before the first part that makes a seam required. Nothing places that step.

3. **LOW.** D13's shed-records lead-in no longer matches the rewritten Writes sub-bullet.
   - Evidence: the lead-in still says "Shed's records cross the cut as bytes at a path the seam names, with shed's codec behind the seam".
   - The new Writes text has the takeover write cross as a fabric-typed patch with "no codec", and the code-tip rewind written by shed's adapter. Neither is fabric handling bytes through the codec.

4. **LOW.** The builder's nil rationale reads against the new installer text.
   - Evidence: the nil reason is still "since fabric's own tests may install neither the real builder (D12) nor a stand-in for it".
   - The new paragraph has fabric's hubforge-built tests run with shed's real adapter and the real stencil judge, "which D12 counts as the mandated fixture path". By the same rule, hubforge could install the real builder too.
   - The record never says why the builder alone stays out of the installer. The stated reason now reads as one that also bars the shed and stencil adapters, which the record admits.

5. **LOW.** The context paragraph after the new text still says flows needing shed's codec or the stencil judge have their seam tests "in the supplying module's package", and that `internal/fabricengine` tests them "as action-kind tests over bytes and paths, which call no codec or judge".
   - Under the new text, any fabric tier1 test built through hubforge has the real codec and judge installed, so a fabric seam test that runs the executor on such a flow would call them.
   - The two statements now need reconciling: either fabric's flow tests clear those seams, or the sentence admits the call.

Tool calls: none beyond my context.
