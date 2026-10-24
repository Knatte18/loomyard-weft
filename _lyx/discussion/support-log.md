# Support log — every lens fork report is kept verbatim in the run records

## Interview

A darn run holds no interview: the writer read the board entry and the code and made every call itself.

## Rejected alternatives

- A `lens-reports/` subdirectory per host: Plan-Write's rotation moves only top-level files, so the reports would outlive their generation, and the rotation would need a directory move it avoids by design.
- A `lens-report-<lens>.md` prefix per directory without the record's stem: a round's reviewer and fixer fans could share a lens name and collide in one run directory.
- Go writing the reports from the shuttle transcript: forks' final messages are only in the writer's session, and parsing transcripts would tie the feature to one provider.
- Declaring the naming in `planparser` or `lenses`: `burlerengine` would depend on a plan package, and `lenses` would pull `configengine` into `planparser`.
- Passing the specs directory to `burlerengine.New` as a parameter: the geometry builders already own the specs directory, and the constructor's told seams are reserved for values no geometry carries.
- A `loom-` spec name: Burler runs standalone too, and the stencils already have a `twill` family for the lens review.

## Review rounds

_No rounds yet._

## Question ledger

- Where does a Discussion-Write report sit, given its lens review is a support log section? Self-pick: beside the support log, named from its stem.
- Does a fork with no report get a file? Self-pick: yes, with the no-report outcome, so an audit sees the gap.
- Should a redone Burler round archive the stale reports? Self-pick: no, out of scope; recorded as an open risk.
