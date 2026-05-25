# ALE Front-Matter Schema Definition

This document establishes the formal contract for the metadata of knowledge granules within the Autopoietic Loop Environment. Every `.md` file in the system must begin with a YAML block consistent with the set rigor level.

## 1. Base Schema (Mandatory for `solid` level)

All standard granules (facts, decisions, rationales) must implement these minimum fields:

```yaml
id: "STR_UNIQUE_UUID_OR_KEY"
type: "knowledge | decision | assumption | rationale"
status: "active | superseded | needs_review | obsolete"
rigor: "solid"
created_at: "YYYY-MM-DD"
updated_at: "YYYY-MM-DD"
tags: []
references: [] # Array of IDs of cross-referenced granules
```

## 2. Mandatory Extension for `open_item`

If an operational output does not pass the validation threshold, it assumes the nature of an `open_item`. The `validation` field changes to `open`, requiring the specification of the following attributes in the front-matter:

```yaml
validation: "open"
priority: "critical | minor | peripheral"
current_assumption: "Concise text of the current operational assumption"
provisional_source: "Reference to the low-validation source"
validation_condition: "Action or evidence required for closure"
impact_on_invalidation: "Cascading effect on the system in case of failure"
dependencies: [] # Array of IDs of child granules depending on this element
```

## 3. Tracking Fields for Review States (`needs_review`)

When the propagation automation or an upstream invalidation impacts a child granule, the file status changes and must include the origin of the change for auditability:

```yaml
status: "needs_review"
impacted_by: "ORIGIN_GRANULE_ID" # Points directly to the cause of the invalidation
```

## 4. Mandatory Extensions for `surgical` Level

If the entire ALE is configured to `rigor: surgical`, every single validated granule extends the base schema by requiring the signature of the involved human operators:

```yaml
rigor: "surgical"
reviewer: "HUMAN_OPERATOR_ID"
review_date: "YYYY-MM-DD"
last_formal_audit: "YYYY-MM-DD"
```

## 5. Superoutput Schema (Software Release)

The final aggregation file (Superoutput) does not follow the schema of single living granules. Instead, it assumes a structure comparable to a software release:

```yaml
version: "X.Y.Z"
released_at: "YYYY-MM-DD HH:MM:SS"
released_by: "HUMAN_OPERATOR_ID"
system_status: "full | partial | degraded"
rigor: "quick | solid | surgical"
included_outputs:
  - id: "GRANULE_A_ID"
    version_snapshot: "v2"
  - id: "GRANULE_B_ID"
    version_snapshot: "v1"
```
