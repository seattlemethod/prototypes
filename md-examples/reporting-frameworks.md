---
id: urn:uuid:{{80c870cc-80c1-4128-9d8f-20f51dbfe1be}}
type: DataBook
title: "Reporting Frameworks"
version: 0.1.0
created: {{2026-06-14}}
modified: {{2026-06-14}}
authors:
  - name: "Charles Hoffman, CPA"
    email: "charles.hoffman@me.com"
license: CC-BY-4.0
description: >
  This is a working proof of concept which provides a machine readable lise of financial reporting frameworks that is also human readable.
tags:
  - financial
  - reporting
  - xbrl
provenance:
  source: "Charles Hoffman, CPA"
  method: "Data was created manually"
manifest:
  entrypoints:
    - block: dataset
    - block: shapes
    - block: queries
---

# {{REPORTING FRAMEWORKS}}

This is a working proof of concept which provides a machine readable lise of financial reporting frameworks that is also human readable.

---

## 📦 Dataset

```turtle {#dataset}
@prefix ex: <http://example.org/> .
@prefix schema: <https://schema.org/> .

ex:item1 a schema:Thing ;
  schema:name "Accounting Equation" ;
  schema:identifier "https://raw.githubusercontent.com/seattlemethod/ae/refs/heads/main/ae-theory.xsd" .

ex:item2 a schema:Thing ;
  schema:name "SFAC6" ;
  schema:identifier "https://raw.githubusercontent.com/seattlemethod/sfac6/refs/heads/main/sfac6-theory.xsd" .

ex:item3 a schema:Thing ;
  schema:name "SFAC8" ;
  schema:identifier "https://raw.githubusercontent.com/seattlemethod/sfac8/refs/heads/main/sfac8-theory.xsd" .

ex:item4 a schema:Thing ;
  schema:name "Common Elements" ;
  schema:identifier "https://raw.githubusercontent.com/seattlemethod/common/refs/heads/main/common-theory.xsd" .

ex:item5 a schema:Thing ;
  schema:name "OCC" ;
  schema:identifier "https://raw.githubusercontent.com/seattlemethod/occ/refs/heads/main/occ-theory.xsd" .

ex:item6 a schema:Thing ;
  schema:name "MINI" ;
  schema:identifier "https://raw.githubusercontent.com/seattlemethod/mini/refs/heads/main/mini-theory.xsd" .

ex:item7 a schema:Thing ;
  schema:name "PROOF" ;
  schema:identifier "https://raw.githubusercontent.com/seattlemethod/proof/refs/heads/main/proof-theory.xsd" .

ex:item8 a schema:Thing ;
  schema:name "AASB 1060" ;
  schema:identifier "https://raw.githubusercontent.com/seattlemethod/aasb1060/refs/heads/main/aasb1060-theory.xsd" .

ex:item9 a schema:Thing ;
  schema:name "IFRS for SMEs" ;
  schema:identifier "https://raw.githubusercontent.com/seattlemethod/ifrs-smes/refs/heads/main/ifrs-smes-theory.xsd" .
