---
round: 3
focus:
  - "internal/loomcli/wiring.go and internal/burlerengine/profile.go: settle whether each review segment's assistant await cap is resolved through shuttleengines.KeepWarm for that segment's own chair strand role (the row's role_prefix joined with ReviewRolePrefix), so a keep_warm_cap_roles entry for the chair's real role reaches the assistant's bound, and whether the role-forming rule lives in one place used by both the profile and the wiring."
  - "internal/loomcli/wiring.go and its wiring test: settle whether a keep_warm_cap_roles entry for one segment's chair role changes only that segment's Assist.Timeout, and whether an entry for the bare role burler-review changes none, with each segment's role prefix declared once as a constant rather than a repeated literal."
  - "internal/loomengine/config.go knob-validation loop: settle whether the refusal message and its comment state the correct zero meaning for each timeout knob, in particular assist_reviewer_timeout_min versus the knobs that defer to shuttle's run_timeout_min, while the negative-value refusal text the config test asserts stays."
---

Focus follows the two open items; the rest of the diff is settled.
