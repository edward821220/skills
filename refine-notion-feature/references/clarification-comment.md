# Clarification Comment

Preserve identifiers across later runs. Sort all questions visible in the current audit by `OQ-###`, split them into consecutive batches of at most three questions, and post one comment per batch. Repeat the same `Questions for PM` heading in every comment; stable identifiers provide the ordering, so do not add batch numbers or prose outside the section.

```markdown
## Questions for PM

#### OQ-001 — <decision-shaped title>

**Source:** <spec section or requirement ID>  
**Current interpretation:** <what the source currently implies>  
**Scenario:** <one concrete case exposing the gap>  
**Question:** <one product decision>  
**Recommendation:** <recommended answer and short reason>  
**Impact:** <behavior, scope, estimate, or ticket boundary affected>

#### OQ-002 — <decision-shaped title>

...
```

End the comment after its third question, or after the final question when fewer than three remain. Start the next batch in a new comment with the same `## Questions for PM` heading. Include no other sections or prose in any comment.

Before posting or retrying, scan every feature comment and PM reply to build one global identifier registry. On resumption, treat a clear PM reply as the answer to the referenced identifier. When the mapping is unclear, reply in the discussion containing that identifier when possible. New questions continue the global identifier sequence; never recycle an identifier. If a batch create partially succeeds or fails ambiguously, reload comments and post only identifiers that are still missing.
