# Fixing code review round 1 of config-keys-nested

F1 asked for a repo-wide sweep of bare config key names with no tool that finds them.
A grep over the underscored template keys was the only practical method;
it missed single-word keys such as `engine` and `scoper`, and any key that ended a string literal, so messages were reported done while several were not.
The self-check forks caught them.
A guard test that scans Go string literals for a template's key names without their module would find this class mechanically.
