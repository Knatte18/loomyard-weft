<!-- This is the decide step of the Bouncer's escalation brief for a round at or past its perch's hard cap.
     renderEscalation picks it over bouncer-fragment-decide-open, fills its marker {{.slug}} and passes the result as the brief's decide_options marker, after the step's number.
lyx-stencil: sha256=87dcd11b9482a8af57759f2bf35060d65591c49a94b64354584fa456317a8aa2 build=072e2ed86cca361e439c0062b67634777d1b3410 time=2026-10-13T07:25:55Z -->

Record your decision on the segment with the one option this round has.
This round is at or past the perch's hard cap, so the segment can only be accepted:
   - `lyx loom circling accept {{.slug}}` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
