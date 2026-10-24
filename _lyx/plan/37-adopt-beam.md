# Card 37 — adopt-beam

**Edit:**
- `internal/adoptcli#namedEntries`
- `internal/adoptcli#takeOverSlug`
- `internal/adoptcli#TestAdopt`

**Extends:**
- `internal/adoptcli/doc.go`

**Intent:**
D8: `lyx adopt` admits a project whose `beam` is true as it admits a started task, taking over its owner and adding its pair when its branches are on origin; adopting a beam moves the beam only, its members keeping their own owners (PATTERN-owner-machine).
`namedEntries` admits a beam project (`OwnerEntry.IsProject` and `Beam`, card 11), and `--all` names every open beam with an owner beside the started tasks; a project whose `beam` is false is refused naming `lyx beam create <slug>` in place of the note refusal, which keeps naming `lyx board promote`.
`takeOverSlug` adds a beam's missing pair through the same `Add` from the prime, which adopts the live branches and their origin record, beam mark included.
`TestAdopt` gains rows: a named beam is adopted with its pair added and its owner written, a grouping project is refused naming `lyx beam create`, and `--all` includes an owned beam.

**ImpactSummary:** Adopt's entry admission widens to beam projects; tasks and the prime are adopted as before.
