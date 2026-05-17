---
status: done
tags:
date: 2026-01-23
---
# Conceptual Design

**Entity-Relationship Diagrams**

SQL
- QUEL ->
- SeQUEL ->
- SQL
	- structured query language

implementations
- mysql (this course)
- oracle
- sql server
- PostgreSQL

# ER Diagrams

- original uses Chen's notations
- this class uses arrow notation + cardinality notation
- crow foot notation (more suited for relational diagrams)

## Entity Set

- groups together same types of data
- presented as a rectangle
- all sets -> no duplicates
- each element has attributes

**Candidate keys**: keys that can serve as a primary key
**Minimal**: One of the key set can not be removed or else they can no longer serve as a primary key
**Primary key**: Candidate key that is enforced by the DBMS (every set must have PK)

### Integrity Contraints
-> **Primary Key**
-> **Domain Constraints**: Included in diagram
-  types 
	- Int
	- float
	- strings
		- char(n) where n is fixed size
		- varchar(n) variable length string on max size n
	- text
	- blob (multimedia)
-> **Assumptions**
-> **Cardinality constraints**

### Relationships

establish link between any two sets

**types**:

- many to many
- one to many
- one to one 