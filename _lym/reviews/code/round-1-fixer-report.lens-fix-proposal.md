```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals F1 and F2, both fixed as proposed, by building cases where each is wrong. I made no further tool calls.

**Findings**

1. **LOW, F1.** The wave-1 text "one search per changed symbol for its callees and the paths that reach it from another process" asks for a call that has no arguments yet.
   - Callees of a symbol are found by reading its body, and the body is only known once the definition has been read.
   - In wave 1 the definition read is issued together with this call, so a callees search cannot name what to look for.
   - Wave 2 ("read together every definition and call site the first wave's results point to") is where callees are read.
   - Evidence: line 26, "A call goes to a later message only when its arguments come from an earlier result". Under that rule, the callee part belongs in wave 2.
   - Only the other-process paths search, which can be a grep for spawn and in-process call sites by symbol name, is valid in wave 1.
   - A literal handler either issues a vague grep or breaks line 26.

2. **LOW, F1.** "one search per change for its failure, restart and resume paths" is not a well-formed wave-1 call.
   - Failure, restart and resume paths have no shared search key. They are found by reading the code around the change.
   - A handler must invent a pattern such as `resume|restart`, which gives noisy results.
   - The review's concern was a missing named call, and this wording only satisfies it on paper.

3. **LOW, F2.** "the read of each file the target names, at the definitions of the symbols it cites" in wave 1 cannot be done in the same message as the searches.
   - A bounded read needs an offset and limit, and those come from locating the definitions.
   - In wave 1 the definitions have not been located, so the only available call is a whole-file read. That is the outcome the edit meant to prevent.
   - Evidence: wave 1 is defined as calls issued "together in one message" with arguments known from the target alone.
   - The wording is also ambiguous. The trailing clause sits between commas in a comma-separated list, so it reads as a separate list item.
   - Possible resolution: put the bounded definition reads in wave 2, or allow a wave-1 grep that locates definitions.

The doc.go sentence "a few waves" and the markers and tests the review listed are unaffected by the edits.

**Further tool calls:** none. My context already holds the stencil text, the review and the rubric.
