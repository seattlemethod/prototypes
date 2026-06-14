---
id: urn:uuid:{{GENERATE-UUID}}
type: DataBook
title: "TITLE OF YOUR DATABOOK"
version: 0.1.0
created: {{DATE}}
modified: {{DATE}}
authors:
  - name: "YOUR NAME"
    email: "you@example.com"
license: CC-BY-4.0
description: >
  Short description of what this DataBook contains and why it exists.
tags:
  - example
  - databook
  - semantic
provenance:
  source: "Describe where the data came from"
  method: "Describe how the data was produced"
manifest:
  entrypoints:
    - block: dataset
    - block: shapes
    - block: queries
---

# {{TITLE OF YOUR DATABOOK}}

A brief explanation of the purpose, scope, and intended use of this DataBook.

---

## 📦 Dataset

```turtle {#dataset}
@prefix ex: <http://example.org/> .
@prefix schema: <https://schema.org/> .

ex:item1 a schema:Thing ;
  schema:name "Example Item" ;
  schema:identifier "item1" .
