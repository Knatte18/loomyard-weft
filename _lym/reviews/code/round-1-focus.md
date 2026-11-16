---
round: 1
focus:
  - "Launch and resume in internal/orchcli/start.go, cli.go and internal/shuttleengine: does every hub and numbered orch launch or resume pass --remote-control with the name, and do writers, reviewers, task sessions and Codex orchs get nothing?"
  - "Name template in internal/orchengine/config.go and remotecontrol.go: does the default render `<machine>::<name>` through agentname.FormatMachineAddress, do {machine}, {hub}, {role} and {name} expand, and are an unknown placeholder, an empty template and the switch off handled without a bad launch line?"
  - "Name quoting in internal/shuttleengine/claudeengine/command.go for both shell dialects: can a template result with `[`, `:`, spaces or quotes (for example `ly:orch[neve]`) reach the shell unquoted or split into several arguments?"
  - "Button path in internal/orchcli/remotecontrol.go (rc-on, rc-bar) and creel's remote-control command: does a click type `/remote-control <name>` with the launch-time name into the live orch pane only, fail visibly when no orch strand is live, and avoid typing into a session that already has it on?"
  - "State record in internal/orchengine/state.go and remotecontrol.go: when is it set on, can it go stale on resume, context cycle or fresh session, and is the lit or dim colour read from the right orch's record when several orchs share a reed window?"
  - "reed status bar in internal/reedengine/bar.go, bindings.go and panebin.go: does the RC button appear only for an orch session, change colour by state, and bind its click in the root table without disturbing the existing Alt-key and follow-toggle bindings or geometry pins?"
  - "Unlicensed scope: the internal/orchroute/paths.go and walk.go changes and the creelcli/creelengine delivery changes; does each serve the decision record, or widen behaviour it does not ask for?"
  - "The claims in _lym/landing/summary.md against the diff, and test economy for the contained reedengine tests TestBindingArgvs_BindsTheFiveAltKeysAndTheStatusClickInTheRootTableOnly and TestComposeSwitchCommand against TestPinBindingsLocked_PinsTheToggleLinesOnlyWhenEverythingResolves."
---

The decision record's Open items (config file and defaults, source of the on/off signal, button colour support) are
settled in the diff, so check that each choice is coherent with the record's Design section.
