   ┌──────────────┐
   │  Developer   │
   │   (you)      │
   └──────┬───────┘
          │
          ▼
   ┌──────────────┐         reads & writes
   │    Agent     │ ◄─────────────────────────┐
   │  (any LLM)   │                           │
   └──────┬───────┘                           │
          │                                   │
          │ writes                            │
          ▼                                   │
   ┌──────────────┐  ◄── Proofing 0 ──┐       │
   │    BRAIN     │  ◄── Proofing 1 ──┤       │
   │  (git repo,  │     persistent    │       │
   │   markdown)  │     memory +      │       │
   │              │     self-audit    │       │
   └──────┬───────┘                   │       │
          │                           │       │
          │ ─── filtered egress ────► │       │
          │                                   │
          ▼                                   │
   ┌──────────────┐                           │
   │     KIT      │ ── public artefacts:      │
   │ (open source │    your codebase, docs,   │
   │  / public)   │    deployable services    │
   └──────────────┘                           │
