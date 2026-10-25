```yaml
lens: fix-consistency
outcome: reported
```

Task: check that F2's change set agrees with the sibling decisions, criteria and cross-references around it (fix-consistency lens).

1. MEDIUM. Changed text: D4 line 118, "A first pass with no status file that `refuseStartOwnedElsewhere` refuses exits with the same code and stays a refused start under D5". Untouched sibling: D7 line 223, "A full pass refused as owned elsewhere is neither: dobby sends today's owner notice naming `lyx adopt` instead (D4), never the failing-pass notice."
   Summary: D7 still sends every owned-elsewhere full-pass refusal to the owner notice, but the new D4 sentence sends the no-status first pass to a refused start.
   Evidence: D4 line 106 makes every pass a full pass except a continue pass, and that includes a fresh start's first pass. So D7:223 and D7:220–222 (a refused start gets the refused-start notice and marker) now give opposite handling for the same refusal. D7:223 needs a carve-out for a first pass with no status file.

2. MEDIUM. Changed text: acceptance bullet line 348, "A full pass refused by any refusal that names another machine as owner, … exits with its own code; dobby then pauses the run under the owner, releases its parent lock and sends the owner notice…".
   Summary: the widened bullet now covers the no-status first pass, which D4 now treats as a refused start with no run to pause.
   Evidence: D4:118 (changed) says that pass "stays a refused start under D5", and D5:202 handles it with a refused-start notice and backoff. The bullet's "any refusal" has no exception for a pass with no status file. Line 337's refused-start bullet doesn't say that an owned-elsewhere code from a fresh start lands there either.

3. LOW. Changed text: D4 line 121, "Dobby reads that code from a run it drives as owned elsewhere".
   Summary: the sentence doesn't say how dobby tells a fresh start's pass, which isn't yet a run it drives, apart from a driven run's pass.
   Evidence: D4:156 and D5:198 count runs dobby drives by its state record. A reader has to infer that a first pass with no status file isn't "a run it drives", and that inference is the dividing line between items 1 and 2.

The support log has no twin listing the refusal sites that would need the same change. Its lines 132 and 134 describe the code only generally.

Tool calls:
- One grep over `_lyx/discussion/support-log.md` and the decision record for exit-code and refusal-site wording. I needed it to find twins of the changed text (the support log, and acceptance bullets past line 232) that my context didn't show.
