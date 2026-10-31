---
round: 2
focus:
  - "internal/fabricengine/pushclassified.go, failRejection: when the judgment fetch succeeds but the branch has no configured upstream or the remote-tracking ref is missing, does it still record not_fast_forward and mail a divergence? Settle that only a fetch showing origin holding commits the branch lacks yields that class, with gitrepo.Repo.HasUnpulled folding a non-zero git exit into (true, nil)."
  - "internal/fabricengine/pushclasses_integration_test.go, TestPushFailureClasses: does a row make each failRejection branch fail when broken (failed judgment fetch recorded as other with the rejection line and fetch error joined; missing upstream), and does the other-class notice assert the plain-retry way forward text rather than only the absence of divergence words?"
  - "internal/gitrepo/push_test.go and internal/loomcli/commitrecords_test.go: does any assertion fail if the rejection error text claims a divergence again (contains remote diverged) or loses git's rejection line?"
  - "internal/fabricengine/pushanchored.go, pushanchored_integration_test.go and doc.go: do the file headers, test names and comments still say a rejection surfaces unwrapped while the error now reaches callers through the failed-push error with errors.Is(err, gitrepo.ErrPushRejected) holding?"
---
