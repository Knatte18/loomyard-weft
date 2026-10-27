<!-- This is the decide step of the Bouncer's escalation brief for a round at or past its perch's hard cap.
     renderEscalation picks it over bouncer-fragment-decide-open, fills its marker {{.slug}} and passes the result as the brief's decide_options marker, after the step's number.
lym-stencil: sha256=09f82bf783d34ba7c3c61794d946548d46babc8536d23ba9709ce2b022b3242b build=ef912d4155479098331ad98253e9a38250450e09 time=2026-10-27T07:47:27Z -->

On the ground of the operator decision you applied, record the segment's outcome with the one option this round has.
This round is at or past the perch's hard cap, so the segment can only be accepted:
   - `lym loom circling accept {{.slug}}` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
