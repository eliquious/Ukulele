Ukulele
=======

Ukulele is an unstructured query language with origins in UnQL and common SQL. It features unique capabilities including the handling of real-time data streams and even map/reduce all within a familiar SQL syntax.

- - -

Components
==========

- Databases

  - Databases are used to store related Collections and Streams.

- Collections

  - Collections are persisted streams of unstructured documents. They allow for data analysis to happen after the fact.

- Streams

  - Streams are non-persisted pipelines of unstructured documents. This component only allows for the real-time analysis of data on the stream. Streams offer a wide-range of filtering capabilities as well as real-time document reduction / modification via UDF function definitions.

Syntax
======

For now the main syntactical elements include `CREATE`, `INSERT`, `FORWARD`, `SELECT`, `DELETE`, `SET` and `DROP`.

- - - 

CREATE
======

Examples
--------

```
CREATE DATABASE abc;
CREATE COLLECTION abc;
CREATE STREAM def;
CREATE [UNIQUE|FULLTEXT|SPATIAL] INDEX index_name ON name USING name.id, name.num;
```

Description
-----------

Streams and Collections can be created using the `CREATE` statement. Future versions may include the optional `FROM` and `WHERE` clauses. If it already exists, nothing happens. Functions will be overridden by default.

Global or session variables can be created this way as well. Global variables are signified by a `@` prefix;

Specification
-------------

```
CREATE [TYPE] [NAME];
CREATE [TYPE] [NAME] WITH OPTIONS [...];
CREATE [TYPE] [NAME] FROM [NAME] WHERE [CONDITIONS];
CREATE [TYPE] [NAME] FROM [NAME] WHERE [CONDITIONS] WITH OPTIONS [...];
```

**Indexes** (collections only)

```
CREATE [UNIQUE|FULLTEXT|SPATIAL] INDEX [INDEX_NAME] ON [NAME] USING [EXPR[...]];
```

**Functions**

```
CREATE FUNCTION [NAME] [IF NOT EXISTS] WITH "[CODE]";
```

**Global/Session Variables**

```
CREATE [GLOBAL|SESSION] VARIABLE [NAME] WITH [OPTIONS];
```

- - - 

INSERT INTO
===========

Examples
--------

```
INSERT 1234 INTO abc;
INSERT 3.141592653 INTO abc;
INSERT "This is a string" INTO abc;
INSERT ["this","is","an","array"] INTO abc;
INSERT { type: "message", content: "This is an object" } INTO abc;

INSERT {
  type:"nested",
  content: {
    content: "nested object",
    x:1,
    y: {str:"hi", str2:"there"},
    z:true
  }
} INTO abc;

INSERT { ... } INTO COLLECTION abc;
INSERT { ... } INTO STREAM abc;
INSERT { type: "message_type" } INTO COLLECTION $abc.type;
INSERT { ..., global: @msg_count++ } INTO COLLECTION abc;
INSERT { ... } AS msg INTO COLLECTION decide(msg) FROM abc;
INSERT EACH [{...}, {...}, {...}, {...}] INTO COLLECTION abc;
```

Description
-----------

Syntax for inserting new documents into Collections or real-time Streams.

Values can be any JSON object. If the data destination type is declared then the collection/stream will be created if it doesn't exist. If no type is declared, then an error will be raised if it doesn't exist.

Global variables are referenced by `@`. Documents can be forwarded into dynamicly named collections/streams by prefixing a `$` to the destination.
Destinations can also be decided by a function call.

Specification
-------------

```
INSERT [EACH] [DOCUMENT[...]] INTO [TYPE] [NAME];
INSERT [EACH] [DOCUMENT[...]] INTO [TYPE] $[DOCUMENT.VARIABLE];
INSERT [EACH] [DOCUMENT[...]] AS [ALIAS] INTO [TYPE] [FUNC([ALIAS])];
```

- - - 

FORWARD
=======

Examples
--------

```
FORWARD extend(abc, {time: now()}) INTO STREAM def FROM abc WHERE abc.type.contains("message");
FORWARD {time: now(), type: abc.type, content: abc.content} INTO STREAM def FROM abc WHERE abc.type.contains("message");
FORWARD reduce(data) INTO COLLECTION abc FROM def WHERE abc.var > 5 HOLD seconds(def, 5) AS data;
FORWARD reduce(data) INTO STREAM abc FROM def WHERE abc.var > 5 HOLD rows(def, 5) AS data;
```

Description
-----------

Essentially, the `FORWARD` command creates a new Collection or Stream from an existing one. These new streams can be filtered, reduced or modified from data produced in real time.

Forward is used to generate a filtered and/or modified stream of data. These new streams can be attached to existing collections/streams and are fed data as documents are inserted into the data source as specified by the FROM clause.

All forwarded streams will have a cluster-wide UUID which will allows for real-time modification or deletion of each stream. A specific command will need to be added to allow for the atomic modification of streams while running.

Specification
-------------

```
FORWARD [DOCUMENT] INTO [TYPE] [NAME] FROM [NAME] WHERE [CONDITION[...]];
FORWARD [NON NULL] [FUNC([DOCUMENT])] INTO [TYPE] [NAME] FROM [NAME] WHERE [CONDITION[...]];
FORWARD [NON NULL] [FUNC([ALIAS])] INTO [TYPE] [NAME] FROM [NAME] WHERE [CONDITION[...]] HOLD [PREDICATE] AS [ALIAS];
FORWARD [EACH] [NON NULL] [FUNC([ALIAS])] INTO [TYPE] [NAME] FROM [NAME] WHERE [CONDITION[...]] HOLD [PREDICATE] AS [ALIAS];
FORWARD [EACH] INTO [TYPE] [NAME] FROM [SELECT];
```

- - - 

SELECT
======

Examples
--------

```
SELECT FROM abc;
SELECT { x:abc.type, y:abc.content.x, z:abc.content.x+50 } FROM abc;
SELECT FROM abc WHERE abc.type=="message";
SELECT avg(ppl, "age") FROM ppl WHERE ppl.salary > 50000;
```

Description
-----------

The `SELECT` command allows for data to be gathered from streams real-time or from persisted collections. This command facilitates the aggregation of data through various means such as grouping, ordering, limiting, mapping and reducing.

Joins may be added later.

Specification
-------------

```
SELECT [FUNC([DOCUMENT])];
SELECT [DISTINCT] [DOCUMENT|FUNC([DOCUMENT])] FROM [NAME[...]] [WHERE [CONDITION[...]]]
    GROUP BY [[NAME[ASC|DESC]]|[EXPR[ASC|DESC]]] [HAVING [[EXPR]|[FUNC]]]
    MAP BY [[NAME[ASC|DESC]]|[EXPR[ASC|DESC]]] [HAVING [[EXPR]|[FUNC]]]
    ORDER BY [[NAME[ASC|DESC]]|[EXPR[ASC|DESC]]|[FUNC]]
    [LIMIT [NUM] [OFFSET[NUM]]]
    [UNION|INTERSECT|EXCEPT]
    [HOLD [PRDICATE] AS [ALIAS]]
    WITH OPTIONS [...];
```

- - - 

DROP
====

Examples
--------

```
DROP DATABASE abc;
DROP COLLECTION abc;
DROP STREAM abc;
DROP INDEX abc;
DROP FUNCTION abc;
```

Description
-----------

Drops a particular database, collection, stream, index or function. If it doesn't exist, nothing happens.

Specification
-------------

```
DROP [TYPE] [NAME];
```

- - - 

DELETE
======

Examples
--------

```
DELETE FROM abc WHERE abc.type == "message";
```

Description
-----------

Delete allows for the filtered removal of documents from a collection.

Specification
-------------

```
DELETE FROM [NAME] WHERE [CONDITIONS] [WITH [OPTIONS]];
```

- - - 

SET
===

Example
-------

```
SET date_format TO "YYYYMMDD";
SET max_connections = 500;
SET @msg_count++;
SET value =+ 2;
```

Description
-----------

Sets a particular global or session variable. Global variables are specified by a preceding `@`.

Specification
-------------

```
SET [NAME] [TO] [VALUE|EXPR] [WITH [OPTIONS]];
SET [EXPR] [WITH [OPTIONS]];
```
