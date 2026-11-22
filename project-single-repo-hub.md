# P4 Single-repository hubs: records and board under refs/lym/, the layout known only to fabric

Rendered 2026-11-22 16:25 UTC; every age below is as of then.
Under: [Board](README.md).

[`single-repo-hub`](design-single-repo-hub.md) · Age 3d · Owner `neve`

Groups the entries filed from the Sketch `fabric-single-repo-mode`: single-repo-fabric-mode, single-repo-code-identity.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T5 | **Second hardener campaign on fabric, one-repository layout, with the fabric verbs**<br>• Hardener run on the one-repository fabric paths the first campaign could not drive<br>• `hardener: packages ["./internal/fabricengine","./internal/fabriccli","./internal/gitrepo"]`<br>• `loom: notify_parent_on ["Harden-Bouncer","Harden-Burler-Fix"]` | [`harden-fabric-single-repo-verbs`](design-harden-fabric-single-repo-verbs.md) | hardener | T1 | 19m<br>19m | fabric |
