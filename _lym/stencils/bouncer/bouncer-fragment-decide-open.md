<!-- This is the decide step of the Bouncer's escalation brief for a round below its perch's hard cap.
     renderEscalation picks it over bouncer-fragment-decide-capped, fills its marker {{.slug}} and passes the result as the brief's decide_options marker, after the step's number.
lym-stencil: sha256=9d40c48b2d077f613f132cc5a0414814b78454861ee8fc138bafa9df06a4a24b build=ef912d4155479098331ad98253e9a38250450e09 time=2026-10-27T07:47:27Z -->

On the ground of the operator decision you applied, record the segment's outcome with exactly one of:
   - `lym loom circling accept {{.slug}}` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
   - `lym loom circling continue {{.slug}}` runs one more round.
