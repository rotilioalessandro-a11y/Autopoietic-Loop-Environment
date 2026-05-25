# ALE Open Design Questions

This document formally tracks architectural doubts, encountered logical limits, and unresolved design nodes in the current version of the Autopoietic Loop Environment framework.

---

## [ODQ-001] Changing Rigor Mid-Flight
- **Problem**: Version 0.7 does not support changing the rigor level once the system has started. However, in dynamic contexts (e.g., a `solid` market analysis intersecting a highly strategic risk target that must become `surgical`), freezing the system is inefficient.
- **Proposed Guiding Principle**: Upgrading to a higher rigor must force the entire graph of active granules into the `status: needs_review` state. The ALE-builder will generate retroactive operational tasks to integrate missing front-matter fields (e.g., `reviewer`) and re-execute granularity checks.
- **Status**: *Open*. Missing the specification for computational load and LLM context impact for graphs exceeding 100 granules.

---

## [ODQ-002] "Impact Preview" Fatigue in the Surgical Level
- **Problem**: In the `surgical` level, the impact preview is mandatory even for minimal changes classified as `cosmetic`. If the density of changes is high, the explicit human validation constraint (Human in the loop) can create a cognitive bottleneck, leading the operator to approve commits without real verification.
- **Possible Solutions**:
  1. Introduce an "auto-commit mode for exclusively lexical changes" verified by a separate micro-agent.
  2. Group cosmetic changes into weekly micro-batches instead of blocking the live session.
- **Status**: *Under discussion*.

---

## [ODQ-003] Cross-Reference Graph Explosion and RAG Window Limits
- **Problem**: As the ALE feeds itself, the number of bidirectional relationships between granules grows quadratically. When the *RAG Decoder* queries the knowledge base, loading the entire sub-graph of connected files risks saturating the context window or introducing information noise.
- **Status**: *Open*. Need to test proximity metrics or graph-pruning algorithms based on the relevance of the current operational task.
