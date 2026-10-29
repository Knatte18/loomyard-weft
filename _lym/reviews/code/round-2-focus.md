---
round: 2
focus:
  - "internal/burlerengine/cluster_test.go, TestFirstMutatingGitCommand: does a row fail if the splitShellSegments redirect case (cluster.go) stops skipping an & attached after a redirection operator, as in git 2>&1 commit -m x?"
  - "internal/burlerengine/cluster.go, splitShellSegments redirect case: is a quoted or backslash-escaped digit operand before a redirect, such as git branch \"2\">x, still kept as an operand rather than dropped as a file-descriptor number?"
  - "internal/burlerengine/cluster_test.go, TestFirstMutatingGitCommand: does a row fail if endSegment stops resetting dropNextWord, if the file-descriptor check stops requiring attached digits, or if a backslash-escaped redirect character stops staying in its word?"
---

Focus targets the redirect handling added to the Burler fork git guard and the rows pinning it.
