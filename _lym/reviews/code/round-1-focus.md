---
round: 1
focus:
  - "internal/creelengine/receive.go `trustedSender`: does the match run on the repository name from the sender's origin remote, and is a hub with no origin or a non-GitHub origin never trusted?"
  - "internal/hubregistry/repository.go: does the origin-remote parse handle ssh, https, `.git` suffix, trailing slash and case, and does the failure path return a reason rather than an empty name that could match an empty `trusted_hubs` entry?"
  - "internal/creelconfig/reinterpret.go and internal/configreg/configreg.go: does the reinterpretation rewrite each shortname to the repository it resolves to on the running machine, drop an unresolvable one with a line naming it, and leave an entry already a repository name unchanged so a rerun is idempotent?"
  - "internal/creelcli/hubsverb.go and internal/creelengine/hubs.go: does `lym creel hubs` show the repository name beside the shortname, and say why for a hub with no origin remote?"
  - "internal/creelengine/fold.go, journal.go and receive.go: does the trust stamp on a cross-hub line show the name beside the shortname, and does the journaled inbound record stay readable by existing consumers?"
  - "Doc and contract changes (contracts/orchrole/orch-role.md, pattern/PATTERN-agent-addressing.md, creelconfig/creelengine/creelcli doc.go, template.yaml): do they match the new behavior, and is any `trusted_hubs` shortname wording left behind?"
  - "internal/fabricengine/codebinding.go: does this change belong to the answer key's scope?"
  - "Tests (creelengine/crosshub_test.go, creelconfig/reinterpret_test.go, creelcli/cli_test.go, testkit/creelfake/hubs.go): is each new behavior covered once, and does any test duplicate an existing one?"
---

Seeded from the decision record's Goal (four wanted behaviors) and the diff stat over `ef32f96a..HEAD`.
