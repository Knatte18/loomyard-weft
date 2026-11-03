# Hub-wide config reconcile by an older lym binary strips keys a newer binary wrote

`reconcile-strips-newer-keys` · Task · bug, reconcile, config · running · Describe

Imported from [issue #2](https://github.com/Knatte18/loomyard/issues/2).

Seen (loomyard hub, 2026-10-31): on OSL-1046 the operator set `pull_request: off` in the hub-wide beam config (Bolt commit 7660dc966, 2026-10-30), and `pr_reviewer: opus[medium]` was set in landing.yaml.
After moving to another machine (Neve), whose installed lym was still built from 31d9ae96 (2026-10-28, before the PR-Review row and the beam `pull_request` key existed), the first lym command ran "lym: reconcile hub-wide config for build (31d9ae96c783 production ...)" (Bolt commit dfe254915) and removed both keys from the committed default layer, since that build's templates did not know them. The binary was updated four minutes later, but the settings were gone, and runs went through PR-Review against the operator's setting.

Workaround: restored both keys by hand with `lym config beam --layer default --set pull_request=off` and `lym config landing --layer default --set 'pr_reviewer=opus[medium]'`.

Expected: an automatic reconcile never removes keys from the shared, committed default layer, at least not when the running binary is older than the build that wrote them; unknown keys are kept (as the manual path already reports them under "preserved") and removal stays an explicit `lym config reconcile --layer default --apply`.


_Moved from #907 in the repository before it was recreated._
