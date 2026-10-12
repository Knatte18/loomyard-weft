# Plan-Write: naming the `-weft` flag in a card's prose

The task's decision record is about reading the weft repository, and its Decision 8 bounds the tool's `-weft` flag.
Writing the flag's name in an inline code span in a card's Intent made `lyx loom validate-plan` fail with `card-fabric-reference`, which treats any span containing the fabric spelling as a command that reaches the fabric repo.
The plan now says "the repository flag" and points at the `repositoryFlag` constant instead, which reads worse than the flag's own name and costs the implementer a lookup.

A finding that names a flag rather than a path or a command spelling would be clearer, or the path rule could exempt a span that is a bare `-<flag>` token.
