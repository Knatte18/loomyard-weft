---
round: 2
focus:
  - "internal/reedengine/bar.go `remoteControlShownCondition`: does the condition draw the RC button only where its click can run, meaning the pane carries a usable strand name, the click line is pinned, and the session's `@lym_button_dir` is non-empty, so the click's `cd` and the orch wiring of `lym orch rc-on` never see an empty directory?"
  - "internal/reedengine/bar.go, internal/reedengine/doc.go, internal/reedengine/paneoptions.go (file header and the `strandNamePaneOption` comment) and internal/reedengine/bar_test.go `rcShownWhen`: do they all describe the same draw condition and the same readers of `@lym_strand_name`, the button click and the RC click?"
  - "internal/orchcli/remotecontrol.go (the rc-on verb's RunE and the launch-mark path around `markRemoteControl`) with internal/orchcli/remotecontrol_test.go: is each branch tested, meaning a `ClientAgentPane` failure reported through `output.Err`, the mark landing on the clicked pane's id, and a failed launch mark that is logged and never fails the launch?"
  - "internal/reedengine/remotecontrol_test.go `TestMarkPaneRemoteControlOnVia` against `TestMarkRemoteControlOn`: does any assertion repeat one the other makes?"
  - "internal/orchengine/template.yaml `remote_control` and `remote_control_name` comments, and the `--dir` usage text in internal/slackcli/toggle.go: do they describe what the code now does, an RC button per agent pane and a slack bar directory that is the orch's?"
---
