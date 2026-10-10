---
round: 2
focus:
  - "In _lyx/discussion/decision-record.md D5 and D3, settle whether the drop signature (message prefix `lyx: seed <subtree>` plus all paths under the stencils or specs subtree) can match a commit made by `lyx stencil sync` (`internal/stencilcli/cli.go`, `syncCmd`, through `CommitSeededStencils`), and whether D5's Bound that dropped commits are regenerated on the next pre-run holds for every commit the signature matches."
  - "In D5 and the Bolt push acceptance criteria, settle what lands when a seed commit sits in the same ahead set as the board commit that triggered `Bolt.Push`: does the decided primitive remove the seed commit, and does the stated acceptance outcome follow from it?"
  - "In D1 and D2, settle where the ancestry answer comes from in a hub whose code repo is not loomyard (`cmd/lyx/stencilseed.go` clears `sourceDir` for a consumer repo, so `fabricengine.StencilSource` returns a zero `Source`) and for the specs subtree (`seedStencilsAt` passes `stencilstore.Source{}`), and whether copies there can still refresh after a first stamp."
  - "In D1, the Constraints gitrepo-client-boundary line and the Notes, settle which `gitrepo` read answers is-ancestor (`Repo.IsAncestor` in `internal/gitrepo/ancestry.go`, CLI-bound, versus `Repo.HeadContains`, an object-existence read through `readGoGit`) and whether all three places name the same one."
  - "In D4 and D5 and Open risks, settle what happens on each start verb when a hub-wide config commit stays ahead in Bolt after a failed push: does the fast-forward refuse, does the stamp stay absent, and do the decision text and Open risks agree on how it clears?"
---

Focus targets the sites where a Decision's stated bound or mechanism may not match the existing code it relies on.
