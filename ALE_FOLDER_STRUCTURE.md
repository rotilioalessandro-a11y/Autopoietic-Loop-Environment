# ALE Folder Structure

To allow both LLM agents (machines) and human operators to navigate and execute writing and archiving operations deterministically, the following standard folder structure for the project is defined.

```text
project-root/
├── .gitignore                      # Exclusion of temporary and system files
├── README.md                       # Project cover and introductory documentation
├── ALE_CORE_FRAMEWORK.md           # Specifications and rules of the architecture
├── ALE_FRONTMATTER_SCHEMA.md       # Metadata contracts
├── ALE_OPEN_DESIGN_QUESTIONS.md    # Tracking of framework evolutions
│
├── config/
│   └── ale_builder_instructions/   # Versioning of the ALE-builder prompts
│       ├── v1_initial_prompt.txt
│       └── current_instructions.md
│
├── core_logic/                     # The Logical Development tree
│   ├── scope.md                    # Root node: the Scope of the system
│   ├── needs.md                    # The general Needs identified
│   └── operational_needs/          # Analytical breakdown into Operational Needs
│       ├── OP_NEED_001_market_mapping.md
│       └── OP_NEED_002_pricing_models.md
│
├── knowledge_base/                 # The live granules operable by the RAG Decoder
│   ├── facts/                      # Validated and stable knowledge (operable facts)
│   ├── decisions/                  # Strategic decisions and related rationales
│   └── open_items/                 # Elements with validation: open (assumptions/estimates)
│
├── archive/                        # Synchronous destination for historization
│   ├── superseded/                 # Granules superseded by newer versions
│   └── obsolete/                   # Granules discarded or no longer coherent with the logical model
│
└── superoutput/                    # The consolidated releases of the entire ecosystem
    ├── superoutput_v1.0.0.md       # Static release ready for the RAG Decoder
    ├── superoutput_v1.1.0.md
    └── superoutput_latest.md       # Main file or pointer link to the latest release
```

### Rigid Rules for Movement and Maintenance:

1. **Archive Synchrony**: At the exact moment a granule changes its state to `status: superseded` or `status: obsolete`, the writing tool *must* physically move the file from the `knowledge_base/` folder to the respective folder in `archive/`. The RAG Decoder must never index the `/archive/` folder.
2. **Promotion of Open Items**: When an ALE operational task (or a human-in-the-loop intervention) validates an open item, the file is modified by updating the front-matter (`validation: validated`, removal of provisional extended fields) and physically moved to `knowledge_base/facts/`.
3. **Session Isolation**: Temporary operational tasks are executed in isolated sessions. Only the final validated outputs are deposited into the permanent folder structure.
