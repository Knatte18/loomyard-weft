# Twill judge parts: critical fixer at Plan-Write only

Batch 04-judge-parts recorded `status: done` at head `998b856d80c523a0e98edf69de6da28cc03d7b18`.
It split the judge steps of the Twill directive into two part stencils, the critical-fixer judge and the plain judge, and `Directive` fills exactly one of them into a single judge marker.
Plan-Write renders the critical part and Discussion-Write renders the plain part, so both hosts keep the re-prompt rule and the boilerplate.

The scope guard listed one deviation outside the batch's declared targets: `internal/twilldirective/twilldirective.go`.
It is informational and did not fail the batch.
No warnings came back from `begin-batch` or `record-batch`.
