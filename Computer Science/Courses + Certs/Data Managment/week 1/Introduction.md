---
status: done
tags:
  - intro
date: 2026-01-21
---
# About

- Relational Databases (sql)
	- Database Management System (DBMS)
- NoSql

# Database Management System

- Applications have DBMS's that store data critical to the app operation 
- Difference between data and information?
	- **Data**: 
		- facts
		- ex: john smith is enrolled in cs336
	- **Information**: 
		- Questions for decision making
		- ex: how many students in a class?
- How to spot the difference?
	- table like (data)
	- functional (information)
	- not structured(information)

## Information System

- **mini-world**: simplification of reality so that we only use relevant data for our application ->
- **requirements/ specifications**: Software engineering requirements (Application) + Data conceptual model (Database) ->
- **logical**: tables (relational models) ->
- **physical design**: sql ->
- **transactions**: software and data meet here, communication with dbms, use stuff like {dbc}

## Physical Design

- Files:
	- sequential (txt, etc)
	- random access files (like arrays)
	- index files
		- hash table
		- tree 
			- by default dbms systems use b trees but can be changed to use hash tables
	- dbms will sort by what key you choose, and then perform a search for it 

## DB Models

- Hierarchical database models
	- restrictive and pointer based (error prone)
	- file systems
- Network (Graph) database models
	- issue when tracking edges (error prone)
- Relational database Model
	- uses tables (relations, mathematical)
	- relational algebra (sql) + relational calculus 