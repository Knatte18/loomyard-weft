# Relevant files

- `internal/burlerengine/assist.go` `AssistCopyPath`, `assistPaths`, `assistSeat`, `assistRulesBlock`, `keepAssistReports` — the assistant seat the parent seat mirrors: per-round paths, end marker, copy beside the review
- `contracts/stencils/burler/burler-step-review.md` — the review step whose optional marker carries a consolidation part
- `contracts/stencils/burler/burler-assist-consolidation.md` — the assistant consolidation part the parent part sits beside
- `internal/loomengine/template.yaml` — defaults and comments for the assistant keys and `pr_review_ask_parent`; the new keys go here
- `internal/loomengine/config.go` `Config`, `ConfigSpec` — key fields, local-layer categories and value validation
- `internal/seatengine/table.go` `Seat` — seat kinds; the parent is not a seat
- `contracts/orchrole/orch-role.md` — the orch's PR review and fork rules; the new request section goes here
- `internal/orchcli/notify.go` `NotifyParent` — the creel path a run uses to mail its parent orch
- `internal/loomcli/landingdeps.go` — how PR-Review's parent ask is wired
- `internal/loomcli/decision.go` `decisionCallerRefusal` — the caller guard the answer verb shares with `decision add --by parent`
