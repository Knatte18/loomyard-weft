# Card 20 — seat-names-delete

**Delete:**
- `internal/seatengine#RoleChair`
- `internal/seatengine#RoleAdvisor`
- `internal/seatengine#AdvisorName`

**Intent:** D6's removal: `seatengine.RoleChair`, `RoleAdvisor` and `AdvisorName` go, so no seat can be named `chair` and no code forms `advisor-<n>` (the one surviving `advisor-<n>` is a name card 19's migration writes into a config file, until the operator renames it).
Card 8 rewrote every reader, and the card runs after card 19, the last card that edits code naming them (`DiscussionTable` and its test).
The Verify line checks the acceptance criterion that no production file names `SingleLLMProducer`, `RoleChair` or `AdvisorName`, comments included, which the build alone cannot show.

**Verify:** ! grep -rln --include=*.go -e SingleLLMProducer -e RoleChair -e AdvisorName internal cmd contracts tools | grep -v '_test\.go$'

**ImpactSummary:** Deletes three exported seatengine names with no remaining reader.
