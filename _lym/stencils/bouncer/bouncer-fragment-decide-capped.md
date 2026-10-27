<!-- This is the decide step of the Bouncer's escalation brief for a round at or past its perch's hard cap.
     renderEscalation picks it over bouncer-fragment-decide-open, fills its marker {{.slug}} and passes the result as the brief's decide_options marker, after the step's number.
lyx-stencil: sha256=de3486319851965520bf5f2e89473c7f7fdc198213cf3dfb930b8b5e31613b68 build=0a1f1615c3eaa1e7ef5b4858110482acec6455b4 time=2026-10-16T09:26:28Z -->

On the ground of the operator decision you applied, record the segment's outcome with the one option this round has.
This round is at or past the perch's hard cap, so the segment can only be accepted:
   - `lyx loom circling accept {{.slug}}` ends the segment unconverged on this round.
     The segment passes with its open findings, recorded as unconverged.
