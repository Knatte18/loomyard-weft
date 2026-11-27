---
round: 2
entries:
  - key: review-timeout-warn-literal-key-name
    status: closed
    finding: F1
    reason: "0a4519673 adds describeReviewTimeoutMinKey in keynames.go and uses it in check's Warn, the ConfigSpec map and LoadConfig's positive-value list; the yaml tag is unchanged"
  - key: stale-pr-text-hold-no-way-forward
    status: closed
    finding: F2
    reason: "aef7d694c rewords syncText's Warn in prreview.go to say the review stays open and to name the way forward, and ADescriptionThatYieldsNoTextHoldsTheContinuedEnding asserts it"
---
