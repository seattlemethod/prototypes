---
title: "Example DataBook: People and Pets"
created: 2026-06-18
author: "Charles Hoffman, CPA"
description: "A tiny DataBook showing metadata, data, validation, and queries."
version: 1.0
---

# People and Pets
This DataBook stores a **tiny dataset** about people and their pets.  
It also includes **validation rules** and a **query** to extract information.

---

## 🐾 Data (RDF/Turtle)
```turtle
@prefix ex: <http://example.org/> .
@prefix schema: <http://schema.org/> .

ex:alice a schema:Person ;
    schema:name "Alice" ;
    schema:pet ex:fluffy .

ex:fluffy a schema:Cat ;
    schema:name "Fluffy" ;
    schema:color "Gray" .
