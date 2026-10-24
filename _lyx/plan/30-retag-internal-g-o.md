# Card 30 — retag-internal-g-o

**Edit:**
- `internal/gatecli#`
- `internal/gateslot#`
- `internal/gitexec#`
- `internal/githubclient#`
- `internal/gitkit#`
- `internal/gitrepo#`
- `internal/gitrepo/internal/gitoracle#`
- `internal/haltnote#`
- `internal/helpcli#`
- `internal/hibernatecli#`
- `internal/hibernateengine#`
- `internal/hubforge#`
- `internal/hubgeom#`
- `internal/hubissues#`
- `internal/hubreconcile#`
- `internal/idecli#`
- `internal/ideengine#`
- `internal/impactset#`
- `internal/languageconfig#`
- `internal/lenses#`
- `internal/lock#`
- `internal/logger#`
- `internal/loggerconfig#`
- `internal/lyxcwd#`
- `internal/lyxtemp#`
- `internal/machinename#`
- `internal/mergeresolve#`
- `internal/modelspec#`
- `internal/orchcli#`
- `internal/orchengine#`
- `internal/output#`

**Intent:**
Retag every test file of these packages to its one tier tag (D5), under the overview's `retag-rules` decision, as card 23 states the rules.

**ImpactSummary:** Changes only the build-constraint lines of these packages' test files, splitting a shared helper file where another tier uses it; no production code or caller changes.
