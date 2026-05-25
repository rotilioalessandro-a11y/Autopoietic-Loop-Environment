# Autopoietic Loop Environment (ALE) Core Framework

An Autopoietic Loop Environment (ALE) is a workspace where artificial intelligence builds its own knowledge, validates it, and keeps it updated while you work with it.

Technically, an ALE is a RAG (Retrieval-Augmented Generation) system where data collection and production exceed the human capacity to remember and process them. It is an environment that builds itself from the inside, feeding on the information retrieved and produced, and within which one acts to extract information.

To build ALEs—by far the working mode with the highest capacity to produce value repeatedly over time—an analytical approach is necessary. Its primary characteristic is designing a logical development that will instruct the artificial intelligence to perform the task.

## Rigor

Not all ALEs require the same intensity. An ALE for organizing a local town fair does not need the same structure as a global market model. **Rigor** is a general setting of the ALE that modulates how strict the discipline is regarding validation, granularity, propagation, and tracking.

Rigor is declared once, when opening the ALE, and applies to the entire system. It must be recorded in the front-matter of the superoutput under the `rigor` field.

**Three levels:**

### `quick`
For small ALEs, confined scopes, and single decisions. You start light, without splitting hairs.
- Granularity: large granules. A granule can contain a whole reasoning. Cross-references are not mandatory.
- Front-matter: reduced schema. Only basic fields are mandatory; extensions apply only where truly needed.
- Impact preview: only on `structural` changes.
- Open items: tracked as a simple list, without all the fields of the `open_item` extension.
- Superoutput: explicit but lean push. No formal snapshot of `included_outputs`.
- **Human in the loop: on everything.** Precisely because the form is loose, human judgment is the only defense against drift. Low rigor loosens the structure, never the control.

### `solid`
Default. The regime described in all other sections of this document. It applies unless stated otherwise.

### `surgical`
For high-risk domains, expensive decisions, and models that must hold up over time. **Requires massive human intervention.** Choosing this level means signing up for a significant review workload. It is used only when the scope justifies it.
- Maximum granularity. Bidirectional cross-references always.
- Full front-matter, with additional tracking fields (e.g., `reviewer`, `review_date` on every validated granule).
- Impact preview mandatory even on `cosmetic` changes.
- Human in the loop extended to operations that would be automatic in `solid`.
- Superoutput: push preceded by formal review.
- Open items: formally tracked, with mandatory periodic review even in the absence of changes.

**Changing rigor mid-flight:** not supported in this version of the model. The guiding principle, when implemented, is that upgrading to a higher rigor entails a total review of existing granules to realign them with the new schema. See `ALE_OPEN_DESIGN_QUESTIONS.md`.

## Logical Development

```text
scope
└── need
    └── operational needs
        └── operational tasks (separate sessions, sized not to degrade context)
            └── operational outputs (knowledge, assumptions, rationales, decisions)
                └── superoutput (aggregation of all operational outputs)
                    └── decoder (RAG based on superoutput + knowledge base)
```

Logical development always starts from the scope, translates into a general need, and analytically breaks down into operational needs. Each operational need corresponds to one or more operational tasks, to be performed in separate sessions small enough not to degrade the LLM context. Each task produces an operational output—and operational outputs include knowledge, assumptions, rationales, and decisions, not just visible deliverables. All operational outputs aggregate into a superoutput. A RAG acting as a decoder is built on top of the superoutput and the generated knowledge base.

However, logical development is not enough. Instructions must be integrated with infrastructural elements, following at least the imperatives described below.

## System Construction

- The logical development must be formalized with a specific prompt asking the LLM to produce instructions to generate the ALE according to this logic, declining it appropriately.
- The generation of instructions must be part of an autonomous and preliminary RAG (**ALE-builder**). Its versions must be tracked and made part of the instruction set composing the ALE.
- The knowledge base must be formalized at maximum granularity, so each granule is self-sufficient to be understood.
- Formalized and standardized handoffs between sessions must be produced as part of the knowledge base itself.
- All granules in the knowledge base must be cross-referenced.
- The knowledge base must be archived in machine-operable scopes, according to the structure defined in `ALE_FOLDER_STRUCTURE.md`. Moving a granule to `/archive/` happens simultaneously with the status change to `superseded` or `obsolete`.
- The format of the entire knowledge base is Markdown.
- Every granule in the knowledge base is constantly typed with front-matter, according to the contract defined in `ALE_FRONTMATTER_SCHEMA.md`.
- Crucial decisions are those characterized by strategic uncertainty: here, **human in the loop** is necessary.
- The ALE must have the function of writing and storing the knowledge base.

## Validation Regime

- Every operational output must be qualified against an explicit validation threshold defined in the instructions. The threshold identifies what constitutes a source of high validation value in that specific operational domain.
- Outputs that pass the threshold enter the knowledge base as **operable facts**.
- Outputs that do not reach the validation threshold are archived as **open items**, themselves constituting a native class of knowledge base granule.
- The status of an open item is typed in the MD front-matter as a mandatory field (e.g., `validation: validated | open`) and every open item specifies: current assumption, provisional source, required validation condition, impact in case of invalidation.
- Open items are cross-referenced to the granules that depend on them.

## System Operational Status

- Only open items classified as **critical** block full operability. Those that are **minor** or **peripheral** leave the system conditionally operational.
- The system is fully operational on domains where all granules are validated, and conditionally operational on domains with active open items.
- The superoutput front-matter contains a field `system_status: full | partial | degraded`, calculated based on how many and which open items are active.

## Recursive Maintenance

- The validation of open items is itself an operational need of the system, fully fitting into the decomposition schema as an operational task.
- Each open item validation is a task for the ALE, with the same validation rules. If unreachable, they become tasks for the human in the loop.
- Invalidation of an open item propagates the review to all granules depending on it, transitively until the graph is closed. The propagation follows the **impact preview** mechanism (dedicated section): the LLM produces the complete list of impacted granules and a proposed impact classification (`scoped` or `structural`, never `cosmetic`), the human confirms or overrides. Child granules change to `status: needs_review`, with the `impacted_by` field tracking the origin of the invalidation. Re-validating each child granule is an operational task for the ALE.

**Invalidation of an operable fact:**
A previously validated granule might turn out to be false due to new evidence. Treatment depends on the type:
- **Operational output** (knowledge, decision, rationale, assumption): changes to `status: needs_review`. If re-validation fails, it can degrade to an open item or be archived as `obsolete`.
- **Logical node** whose factual premise was wrong: amended according to the "Amendment of the Logical Model" procedure.
- **Open item previously closed as validated**: reverts to `validation: open` and the validation mechanism restarts.
In all cases, the **impact preview** applies for propagation to dependents.

## Impact Preview

Every operation that changes the state of an existing granule—amendment of a logical node, dropping of an open item, invalidation of an operable fact, re-classification—requires an **impact preview** generated by the LLM before the commit. Never a silent commit, never automatic propagation without visibility.

**Impact preview content:**
- List of impacted granules by transitive propagation, until the dependency graph is closed.
- Expected destination status for each: `needs_review`, `open`, `superseded`, `obsolete`.
- Proposed overall impact classification: `scoped` or `structural` (never `cosmetic` when propagation crosses dependencies).
- 2-3 line justification for the classification.
- Proposed `impacted_by` field for each child granule (see front-matter schema).

**Mechanical criterion for classification:**
The LLM calculates propagation and proposes the classification based on this rule:
- Impact contained within a single operational need → `scoped` proposal.
- Impact crossing multiple operational needs or going back up to a `need` or `scope` node → `structural` proposal.
The proposal is explicitly justified.

**Role of the human:**
The human receives preview + proposed classification + justification. Confirms or overrides. Never starts from scratch. The commit only happens after explicit decision.

**Tracking:**
Every granule changing to `needs_review` due to propagation receives the `impacted_by: id` field in its front-matter, pointing to the granule originating the change. This allows reconstructing propagation chains.

## Amendment of the Logical Model

Amending a logical tree node (scope, need, operational need, task) is an **invasive surgical intervention**. It is done with caution, sparingly, and only if strictly necessary. Every amendment is an open wound in the system's coherence: it should be performed when the cost of not doing it exceeds the cost of propagation.

**Before amending, verify in order:**
1. Is the change truly in the logical model, or can it be resolved at the operational output level?
2. Does an open item already cover the issue, avoiding touching the node?
3. Is the change structural, or is it an aesthetic preference disguised?
If at least one of these alternative paths holds, the amendment **is not done**.

**When amendment is inevitable, these rules apply:**
- Every logical node is a granule with extended front-matter: `version`, `supersedes`, `status: active | superseded | needs_review`, `change_class: cosmetic | scoped | structural`.
- A version is never overwritten: the previous one remains archived with `status: superseded`. History is tracked.
- Impact classification is **human in the loop**, supported by the **impact preview**. The LLM produces the preview + proposed classification + justification; the human confirms or overrides. Never automatic, as this is the point of maximum bias towards "cosmetic".
- Propagation:
  - `cosmetic`: no child impacted.
  - `scoped`: direct children change to `status: needs_review`.
  - `structural`: entire subtree changes to `status: needs_review`.
- Granules in `needs_review` are not automatically deleted or redone: they are marked. The review of each is an ALE operational task.

**Operational outputs under an amended parent task:**
- Operational outputs produced under a previous version of the parent task are **re-labeled**, never deleted.
- Extended front-matter for outputs: `parent_version_at_creation` field, tracking under which version of the parent the output was produced.
- When the parent task is amended (`scoped` or `structural`), child outputs change to `status: needs_review`. The review decides the outcome: `validated_under_new_parent`, to be amended, or `obsolete` (but kept in archive).

**Superoutput Versioning:**
- The superoutput is **versioned like a software release**, not as a live stream.
- Upstream granule state changes accumulate. When the human decides that the critical mass of changes justifies a new release, they execute an **explicit push**: the superoutput is recalculated and becomes a new version.
- Previous versions of the superoutput remain archived. The decoder/RAG always operates on the latest released version.
- The superoutput front-matter includes: `version`, `released_at`, `released_by`, plus the already expected `system_status`.
