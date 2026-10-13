<!-- This is the decide step of the Bouncer's escalation brief for a round below its perch's hard cap.
     renderEscalation picks it over bouncer-fragment-decide-capped, fills its marker {{.slug}} and passes the result as the brief's decide_options marker, after the step's number.
lyx-stencil: sha256=cc232a2e14511824b89c121bb6e57e080a2b90d407d4b19054e458b56cb3107d build=072e2ed86cca361e439c0062b67634777d1b3410 time=2026-10-13T07:25:55Z -->

Record your decision on the segment with exactly one of:
   - `lyx loom circling accept {{.slug}}` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lyx loom circling continue {{.slug}}` runs one more round.
