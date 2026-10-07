# Card 4 — `lyx burler await-review`

**Create:**
- `internal/burlercli/await.go`

**Edit:**
- `internal/burlercli/cli.go`
- `internal/burlercli/cli_test.go`
- `tools/sandbox/SANDBOX-BURLER-SUITE.md`
- `docs/overview.md`

**Uses:**
- `internal/burlercli#burlerCLI.validateReviewCmd`

**Intent:**
Add the wait verb D5 defines: `lyx burler await-review <marker-path>`, the command the fixer runs until the review is ready.
It is a cobra subcommand under `lyx burler` with a `Short`, JSON envelopes and a `Long` with an example (`PATTERN-cli-cobra`), built like `validateReviewCmd`.

Behaviour: a relative marker path resolves against the seam cwd;
the verb polls for that one path and returns as soon as it exists, printing `output.Ok` with `ready: true` and the absolute marker path;
when the cap elapses first it prints `output.Ok` with `ready: false`, telling the agent to call again.
Both answers exit 0;
a missing argument or an unreadable stat (any error but not-exist) is an `output.Err` with exit 1.
The cap is a `--cap` duration flag defaulting to 100s, below the agent's default two-minute Bash tool timeout, and the poll interval is short and fixed (a fraction of a second), so a test shortens the wait through the flag rather than waiting it out.
`resolvePersistentPreRun` skips wiring for the verb as it does for `validate-review`: it needs no hub, mode, git repository or engine.

What the verb lets through, and its bound: it reports ready for any existing path it is handed, so a caller could release itself by naming another file;
it reads one caller-named path and writes nothing, and keeping the fixer to the marker Go writes is the fixer prompt's rule (card 5), a prompt-enforced risk the decision record names.

Tests in `cli_test.go`, from a temp directory outside any git repository: a present marker returns `ready: true` at once;
an absent marker returns `ready: false` after a short `--cap`;
a marker created while the verb polls returns `ready: true`;
the directory's entries are unchanged afterwards (the verb writes nothing).
`SANDBOX-BURLER-SUITE.md` gains a scenario step that runs the verb against an absent and a present file, so the sandbox suite exercises it (`PATTERN-sandbox-coverage`).
`docs/overview.md`'s burler entry lists the verb beside `validate-review`.

**ImpactSummary:** one additive `lyx burler` subcommand and its pre-run wiring skip; no existing verb changes.
