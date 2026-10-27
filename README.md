# basic_ETL_informatica-PowerCenter

# SQL, Data Warehousing & Informatica PowerCenter – Study Notes

Personal notes covering **SQL**, **Data Warehousing concepts**, **Agile/SDLC**, and **Informatica PowerCenter (ETL)**.

---

## Table of Contents

1. [Data Storage Basics](#1-data-storage-basics)
2. [Database & SQL Basics](#2-database--sql-basics)
3. [Working as a SQL / ETL Developer (Data Flow)](#3-working-as-a-sql--etl-developer-data-flow)
4. [OLTP vs OLAP](#4-oltp-vs-olap)
5. [SQL Commands (DDL, DML, TCL, DCL)](#5-sql-commands)
6. [Constraints](#6-constraints)
7. [Clauses & Execution Order](#7-clauses--execution-order)
8. [Errors in SQL](#8-errors-in-sql)
9. [Joins](#9-joins)
10. [Set Operators](#10-set-operators)
11. [Views](#11-views)
12. [Index](#12-index)
13. [Window Functions: RANK / DENSE_RANK](#13-window-functions-rank--dense_rank)
14. [Pseudocolumns](#14-pseudocolumns)
15. [Misc SQL Terms](#15-misc-sql-terms)
16. [Project Workflow & Environments](#16-project-workflow--environments)
17. [SDLC, Waterfall & Agile](#17-sdlc-waterfall--agile)
18. [Data Warehouse Concepts](#18-data-warehouse-concepts)
19. [Dimensions & SCD](#19-dimensions--slowly-changing-dimensions)
20. [Fact Tables](#20-fact-tables)
21. [Metadata / Dictionary Tables](#21-metadata--dictionary-tables)
22. [Informatica & ETL](#22-informatica--etl)
23. [Informatica Transformations](#23-informatica-transformations)
24. [Flat Files](#24-flat-files)
25. [CDC, Variables & Target Load Plan](#25-cdc-variables--target-load-plan)
26. [Removing Duplicates](#26-ways-to-remove-duplicates)
27. [Quick Comparison Tables](#27-quick-comparison-tables)

---

## 1. Data Storage Basics

Data (for example from an API) is stored in two ways:

### a) File format
- **Fixed-width file** – data stored as a continuous string where each field has a fixed width (e.g. `10Apune`).
- **Delimited file** – fields separated by a delimiter (comma, pipe, space, etc.).

### b) Database
- Data is stored in an organized way so it can be accessed easily.
- A **DBMS** (Database Management System) is used to manage a database.
- Examples: MySQL, Oracle, Microsoft SQL Server, IBM DB2, OODBMS.
- **Relational database** – data is stored in the form of **tables**.

---

## 2. Database & SQL Basics

**SQL** (Structured Query Language) is the language used to communicate with a database.

- Non-procedural language (you say *what* you want, not *how*).
- SQL keywords are not case-sensitive; commands cannot be abbreviated.
- Standardised by **ANSI** (American National Standards Institute).

**DBMS vs RDBMS**
- In a plain DBMS there is no relationship between tables (no joins).
- **RDBMS** follows Codd's rules and introduces **constraints** (Primary Key, Foreign Key) so tables can be linked and joined.

---

## 3. Working as a SQL / ETL Developer (Data Flow)

```
Users → OLTP → ODS → (Cleansing, Merging, Scrubbing) → STG → Data Warehouse (OLAP) → Reports / BI
```

- **OLTP** – users continuously use the service and data is stored here (client-server / data storage).
- **ODS (Operational Data Store)** – a replica/dump of current OLTP data, copied *as-is* (no transformation). Needed because OLTP data is only available for a limited time.
- **Cleansing / Merging / Scrubbing** – operations performed on ODS data.
- **STG (Staging)** – a self-use layer holding the stabilised, current data from ODS.
- **Data Warehouse (OLAP)** – built by the developer; used by Java developers / reporting teams to build reports for decision making.

**Data Warehouse** = *Subject-oriented, Integrated, Non-volatile, Time-variant* collection of data used for reporting and decisions.

---

## 4. OLTP vs OLAP

| OLTP | OLAP |
|------|------|
| Online Transaction Processing | Online Analytical Processing |
| Online database modifying system | Online database query management system |
| Contains only current data | Contains historical data |
| Uses standard DBMS | Uses data warehouse |
| Used for business tasks | Used for data mining / analysis |
| Tables are **normalized** | Tables are **not normalized** |
| Small data (MB – GB) | Large data (TB – PB) |

---

## 5. SQL Commands

### 5.1 DDL – Data Definition Language
- Interacts directly with the database; **cannot be rolled back**; fast (auto-commit).
- Commands: `CREATE, ALTER, DROP, TRUNCATE, RENAME, COMMENT, FLASHBACK, PURGE`

```sql
-- CREATE
CREATE TABLE Emp (
    Id    INT,
    Ename VARCHAR(5),
    Eno   NUMBER(5)
);

-- TRUNCATE: removes all data, structure remains
TRUNCATE TABLE Emp;

-- DROP: removes data + structure permanently
DROP TABLE Emp;

-- ALTER: change structure
ALTER TABLE Emp DROP COLUMN Comm;
```

### 5.2 DML – Data Manipulation Language
- Deals only with **data**. Changes first go to a buffer and move to the database on `COMMIT`.
- Slower (buffer), but **rollback is possible**.

```sql
-- INSERT
INSERT INTO Emp VALUES (1, 'a', 1000);
INSERT INTO Emp (Eno, EName, Sal) VALUES (1, 'a', 1000);

-- UPDATE
UPDATE Emp SET EName = 'aaa';
UPDATE Emp SET EName = 'aaa', Sal = 1000;
UPDATE Emp SET EName = 'aaa' WHERE Eno = 1;

-- DELETE
DELETE Emp;
DELETE FROM Emp WHERE EName = 'aaa';
DELETE FROM Emp WHERE EName = 'aaa' AND Eno = 1;
```

### 5.3 TCL – Transaction Control Language
- Valid only for a **transaction** (a set of statements ending with commit or rollback).
- `COMMIT` – make changes permanent.
- `ROLLBACK` – undo work done by DML.
- (`SAVEPOINT` – set a point to roll back to.)

```sql
INSERT INTO Emp VALUES (1, 'a', 1000);
UPDATE Emp SET Sal = 2000 WHERE Eno = 1;
COMMIT;      -- or ROLLBACK;
```

### 5.4 DCL – Data Control Language
- Deals only with **privileges/permissions**.
- Causes an implicit commit before and after the statement; cannot be undone; fast.
- Two kinds of privileges: **System** (admin only) and **Object** (granted to users).

```sql
GRANT  SELECT ON Emp TO u1;
REVOKE SELECT ON Emp FROM u1;
```

### DELETE vs TRUNCATE vs DROP

| DELETE | TRUNCATE |
|--------|----------|
| DML | DDL |
| Rollback possible | Rollback not possible |
| Deletes single or multiple rows | Deletes all rows |
| `WHERE` clause allowed | `WHERE` clause not allowed |
| Space is not released | Space is released |
| Slower (buffer) | Faster |

### ALTER vs UPDATE

| ALTER | UPDATE |
|-------|--------|
| DDL | DML |
| Add / delete / modify attributes (columns) | Update data |
| Changes **structure** | Changes only **data** |
| New column values are `NULL` by default | Sets specified value in the table |

---

## 6. Constraints

Rules and restrictions applied on table data to ensure **accuracy and reliability**.

`CHECK | NOT NULL | UNIQUE | PRIMARY KEY | DEFAULT | FOREIGN KEY | INDEX`

| Constraint | Meaning |
|-----------|---------|
| **CHECK** | Value must satisfy a given condition |
| **NOT NULL** | Column cannot contain `NULL` (NULL = missing/undefined, not zero or empty) |
| **UNIQUE** | All values in the column must be different (allows NULL) |
| **DEFAULT** | Sets a default value when none is given |
| **INDEX** | Speeds up read operations |
| **PRIMARY KEY** | Identifies each record uniquely; only **one** per table; no NULLs; unique |
| **FOREIGN KEY** | Links two tables; matches the primary key of one table with a column of another; can contain NULLs / duplicates |

**Primary key operations**
```sql
-- Add PK to an existing column
ALTER TABLE emp ADD CONSTRAINT pk_emp_id PRIMARY KEY (id);

-- Remove PK
ALTER TABLE emp DROP CONSTRAINT pk_emp_id;
```

**Data Integrity** – data is correct and consistent and follows the rules.

**Auto Increment** – automatically generates the primary key so you don't insert it manually.

---

## 7. Clauses & Execution Order

Written order: `SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY`

**Logical execution order:**

```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY
```

```sql
-- DISTINCT: returns only unique values
SELECT DISTINCT column_name FROM table_name;

-- WHERE: filter rows
SELECT column_name FROM table_name WHERE age > 20;

-- ORDER BY: sort ascending / descending
SELECT * FROM user ORDER BY age;

-- GROUP BY: group rows with the same values into summary rows
-- HAVING : filter groups using aggregate functions
```

---

## 8. Errors in SQL

1. **Syntax errors** – statement doesn't follow correct syntax/structure.
2. **Semantic errors** – grammatically correct but gives the wrong result due to incorrect logic.
3. **Constraint violation** – statement violates a constraint on the database.
4. **Transaction errors** – transaction fails (e.g. locking problems).

---

## 9. Joins

Combine records from two or more tables.

```sql
-- INNER JOIN: matching rows in both tables
SELECT column_name
FROM table1
INNER JOIN table2 ON table1.column_name = table2.column_name;

-- LEFT JOIN: all rows from left + matching rows from right
SELECT column_name
FROM table1
LEFT JOIN table2 ON table1.column_name = table2.column_name;

-- RIGHT JOIN: all rows from right + matching rows from left
SELECT column_name
FROM table1
RIGHT JOIN table2 ON table1.column_name = table2.column_name;

-- FULL OUTER JOIN: all rows when there is a match in either table
SELECT column_name
FROM table1
FULL OUTER JOIN table2 ON table1.column_name = table2.column_name;

-- SELF JOIN: table joined to itself (alias required)
SELECT a.column_name, b.column_name
FROM table1 a, table1 b
WHERE a.common_field = b.common_field;

-- CARTESIAN JOIN: cartesian product of the tables
SELECT table1.column1, table2.column2
FROM table1, table2;
```

---

## 10. Set Operators

Combine results of two or more `SELECT` statements (same number of columns & compatible data types).

| Operator | Description |
|----------|-------------|
| `UNION` | Combines result sets, **removes duplicates** |
| `UNION ALL` | Combines result sets, **keeps duplicates** |
| `MINUS` | Rows in the first query but not in the second |
| `INTERSECT` | Common rows of both queries (no duplicates) |

```sql
SELECT * FROM t1 UNION      SELECT * FROM t2;
SELECT * FROM t1 UNION ALL  SELECT * FROM t2;
SELECT * FROM t1 MINUS      SELECT * FROM t2;
SELECT * FROM t1 INTERSECT  SELECT * FROM t2;

-- Unique values with UNION ALL
SELECT DISTINCT * FROM (
    SELECT * FROM t1
    UNION ALL
    SELECT * FROM t2
);
```

---

## 11. Views

A **view** is a virtual table (logical subset of data from one or more tables). Used for **security**, **data restriction**, and minimizing hits on base tables.

```sql
CREATE VIEW Emp_v AS
SELECT id, name, address FROM t1;
```

| Type | Description |
|------|-------------|
| **Simple view** | Created from a single base table; holds no data |
| **Complex view** | Created from more than one table (joins) |
| **Materialized view** | Physical copy that stores query result; refreshed using `DBMS_MVIEW.REFRESH` |
| **Force view** | Created even if the base table doesn't exist yet (`CREATE FORCE VIEW`) |

```sql
-- Complex view
CREATE VIEW Emp_c AS
SELECT t1.id, t1.name, t2.cid
FROM t1, t2
WHERE t1.id = t2.cid;

-- Materialized view
CREATE MATERIALIZED VIEW Emp_m AS
SELECT * FROM t1;

-- Force view
CREATE FORCE VIEW Emp_f AS
SELECT * FROM t1;
```

---

## 12. Index

A schema object used to **speed up retrieval of rows** using pointers. Good for `SELECT`; slower for `INSERT / UPDATE / DELETE`.

Attributes: access type, access time, insertion time, deletion time, space overhead.

| B-Tree Index | Bitmap Index |
|--------------|--------------|
| Default index | Based on a bitmap / truth table |
| Tree-based | Best for columns with few distinct values |
| Efficient for range queries | Efficient for heavy analytical queries |
| Uses less space | Uses more space |
| Used when data is unique | Used when duplicates are present |

---

## 13. Window Functions: RANK / DENSE_RANK

```sql
RANK()       OVER ([PARTITION BY expr] ORDER BY expr [ASC|DESC]);
DENSE_RANK() OVER ([PARTITION BY expr] ORDER BY expr [ASC|DESC]);
```

| RANK | DENSE_RANK |
|------|------------|
| Ties get the same rank; **next rank is skipped** (gaps) | Ties get the same rank; **no gaps** |
| Data `[10,20,20,30,40]` → `[1,2,2,4,5]` | Data `[10,20,20,30,40]` → `[1,2,2,3,4]` |
| Non-sequential ranks | Sequential ranks |

---

## 14. Pseudocolumns

Behave like table columns but are **not stored** in the table. You can `SELECT` them, but not insert/update/delete.

1. **ROWID** – permanent unique identifier (physical address) of a row; static; used to remove duplicates.
   ```sql
   DELETE FROM t1
   WHERE ROWID NOT IN (SELECT MAX(ROWID) FROM t1 GROUP BY c1, c2);
   ```
2. **ROWNUM** – temporary sequential number assigned as rows are returned; changes with the query; dynamic.
   ```sql
   SELECT ROWNUM, t1.* FROM t1 WHERE sal > 10000;
   SELECT * FROM t1 WHERE ROWNUM <= 10;
   ```
3. **LEVEL** – hierarchy key; starts at 1 and increases by 1. Used to generate rows.
   ```sql
   SELECT LEVEL FROM dual CONNECT BY LEVEL <= 100;
   ```

| ROWID | ROWNUM |
|-------|--------|
| Unique identifier of each row | Temporary value; changes with query |
| Quick access / search | Returns a specific number of rows |
| Has physical address | Dynamic |
| Static, unique | Can be repeated |

---

## 15. Misc SQL Terms

- **CHAR** – fixed-length character string.
- **VARCHAR2** – variable-length character string.
- **Normalization** – organizing data to minimize redundancy and avoid insert/update/delete anomalies (removes partial & transitive dependency).
- **De-normalization** – deliberately adding redundancy to improve **read** performance (at the cost of write performance).
- **Query** – retrieves required data; optimise for best performance.
- **Subquery** – a query embedded inside another query (e.g. in `WHERE`).
- **Operators** – Arithmetic, Logical, Comparison.
- **Table** – basic unit of storage, made of rows and columns.
- **Sequence** – generates numeric values.
- **Synonym** – alternative name for an object.
- **Server** – computer system that stores data and runs applications over a network.
- **Domain** – set of values that can be stored in a column.

---

## 16. Project Workflow & Environments

**Common terms**

| Term | Meaning |
|------|---------|
| BA | Business Analyst |
| FBA / TBA | Functional / Technical Business Analyst |
| HCD / LCD | High-level / Low-level Code Document |
| IA | Impact Analysis document |
| TD | Technical Design document |
| KT | Knowledge Transfer |
| Staging | Landing layer |
| Bitbucket | Version control – track and control code |
| JIRA | Ticketing tool – assign, track and control work |
| UTR | Unit Test Review |
| AMS | Application Maintenance Support team |

**Flow:** `Client → BA (HCD/LCD) → DEV (IA, TD) → Testing → Production lineup → Support (AMS)`

**Environments:** `DEV → QA (testing, historic data) → Pre-Prod → PROD (current data for client)`

**Layers in DEV:** `OLTP → ODS → STG → DW` – code is stored in Bitbucket (as XML) for testing.

**Testing types:** Functional testing, UTR, SIT, CIT, UAT, RIT.

**After go-live – two kinds of problems:**
1. **CR (Change Request)** – new business requirement from client, handled by BA.
2. **Defect** – technical issue.

> Before changing a table/column name, do an **impact analysis** on dependent objects. No impact analysis is needed for new development – a Technical Design doc is created instead.

---

## 17. SDLC, Waterfall & Agile

**SDLC (Software Development Life Cycle)** – multiple components of software developed simultaneously within a deadline.
Methods: Waterfall, Agile, Spiral, Scrum, Lean (Waterfall and Agile are the most used).

### Waterfall
- Traditional, rigid, sequential method; client not continuously involved.
- **Flow:** Requirement → Design → Implementation → Test → Deploy → Maintenance
- **Benefits:** easy to manage, good for small projects, clearly defined stages, well documented.
- **Drawbacks:** inflexible, no mid-process changes, testing is delayed, low chance of satisfying client, resources sit idle, not suited for big projects.

### Agile
- Continuous implementation and integration; overcomes waterfall's drawbacks.
- Work is divided into short cycles (**sprints** of 15 / 21 / 30 days) with daily review.

| Term | Meaning |
|------|---------|
| **Sprint** | Fixed time-box of work (15/21/30 days); core unit of Scrum |
| **Sprint Planning** | BA distributes the project into sprints (design, dev, test, production) |
| **Scrum Master (SM)** | Facilitator / project manager for the agile team (technical or non-technical) |
| **Story Point (SP)** | Estimate of effort; e.g. 2 SP ≈ 2 days |
| **Backlog** | Pending work not yet done / carried to the current sprint |
| **Story assign** | Tasks assigned by the Scrum Master and tracked easily |
| **Priority** | Set by Scrum Master while picking requests/requirements |
| **Retrospective call** | After a sprint, SM, TL and team discuss what succeeded and failed |
| **Epic** | Large story divided into smaller stories |
| **Zero Sprint** | Pre-planning before the first sprint |
| **Spike** | Used when requirements are unclear |
| **Velocity** | Amount of work a team delivers in one sprint |

### Agile vs Waterfall

| Agile | Waterfall |
|-------|-----------|
| Shorter development cycle | Longer development cycle |
| More flexible | Less flexible |
| Product delivered every 2–4 weeks | Product delivered after a long time |
| Best for large projects | Best for small projects |
| Reviewed daily | Reviewed weekly |
| Self-governing teams | Hierarchical teams |
| Focus on business value | Focus on client requirements |

---

## 18. Data Warehouse Concepts

- **Data Warehouse** – Subject-oriented, Integrated, Non-volatile data from multiple sources; used for reports and business decisions.
- **Metadata** – data about data; the road map of the warehouse.
- **Data Cube** – represents data in multiple dimensions.
- **Data Mart** – subset of organization-wide data.
- **Virtual Warehouse** – a view over an operational data warehouse.
- **Data Grain** – level of detail at which data is stored; the combination of columns that makes each record unique.
- **Data Granularity** – how much detail is stored (e.g. time series, order transaction).
- **ER Model** – blueprint of a database showing how data elements relate (ER diagram, data flow diagram).
- **Data Model** – visual representation of an organization's data. Types: Logical, Physical, Conceptual, Relational, Hierarchical, Network.
- **Data modeling tools** – Erwin, DbSchema, MySQL Workbench, IBM tools, Oracle SQL Developer.
- **Schema** – logical representation of a database. Types: Star, Snowflake, Galaxy / Fact Constellation, etc.
- **Tablespace** – storage area for tables; dropping a tablespace removes all its tables quickly.

### Star vs Snowflake Schema

| Star | Snowflake |
|------|-----------|
| Fact + dimension tables | Fact + dimension + sub-dimension tables |
| Top-down model | Bottom-up model |
| Uses more space | Uses less space |
| Less execution time | More execution time |
| Not normalized | Normalized |
| Simple design | Complex design |
| Fewer foreign keys | More foreign keys |

### Fact vs Dimension

| Fact Table | Dimension Table |
|------------|-----------------|
| Contains measures | Contains descriptive attributes |
| Fewer attributes | More attributes |
| More records | Fewer records |
| Vertical (deep) table | Horizontal (wide) table |
| Numeric (and text) data | Text data |
| Created after dimensions | Created before facts |

### Grain vs Granularity

| Grain | Granularity |
|-------|-------------|
| Individual unit | Detail in small parts |
| Basic unit of data | Represents process / dataset / system |
| Smallest unit represented | Level of detail |

### Data Integration terms
- **Data Integration** – combine data from multiple sources in a unified format.
- **Data Cleaning** – make data accurate and reliable.
- **Data Governance** – enforce data policies and regulatory compliance.
- **Data Transformation** – turn data into meaningful information.
- **Data Analysis** – analyse data for informed decisions.

---

## 19. Dimensions & Slowly Changing Dimensions

A **dimension** holds descriptive attributes (parent-child hierarchies) and is used to maintain historical data.

**Surrogate key** – an artificial unique key (doesn't exist in the real world) used as primary key when no natural primary key exists.

**Types of dimensions:** Conformed, Junk, Slowly Changing, Degenerate, Late-arriving, Rapidly changing, Stacked, Inferred, Role-playing, Static, Shrunken.

1. **Conformed** – created once and reused across one or many data marts / warehouses.
2. **Junk** – a single table combining several low-cardinality flags/attributes to avoid many foreign keys.
3. **Degenerate** – dimension attribute kept in the fact table (e.g. invoice number); no separate table.
4. **Late-arriving** – when a dimension record isn't available yet, load facts with a placeholder and update the dimension when it arrives.
5. **Slowly Changing Dimension (SCD)** – attributes that change slowly over time; uses a surrogate key as primary key.

| SCD Type | Behavior |
|----------|----------|
| **Type 1** | Overwrites old value with new; **no history** (main drawback) |
| **Type 2** | Keeps **full history**: new row for each change; uses start date / end date (or active flag) to mark current vs old; data volume grows |
| **Type 3** | Keeps **partial history** (limited) by adding a column for the previous value (e.g. Thane → Nashik → Pune) |

---

## 20. Fact Tables

Contain **measures** (additive, semi-additive, non-additive) and **foreign keys** to dimensions.

| Type | Description |
|------|-------------|
| **Transaction** | Records individual events; most granular |
| **Snapshot** | Captures data at regular intervals (daily / weekly / monthly) |
| **Accumulating** | Tracks progress through multiple stages of a business process lifecycle |
| **Factless** | Contains only keys, no measures |
| **Historical** | Stores historical facts |

---

## 21. Metadata / Dictionary Tables

Used to quickly find information (e.g. impact analysis) instead of searching code line by line.

| Table | Stores |
|-------|--------|
| `ALL_TABLES` | Table info – owner, creation date, records, permissions |
| `ALL_CONSTRAINTS` | Constraint info – owner, permissions, names |
| `ALL_VIEWS` | View info – permissions, owner |
| `ALL_INDEXES` | Index info |
| `ALL_SOURCE` | Procedures, triggers, functions – all code |
| `ALL_TAB_COLUMNS` | Column info – used to find/change column names quickly |
| Logs / Error tables | Error & success messages (error line, function, trigger) |

```sql
SELECT * FROM all_tables WHERE table_name LIKE '%a%';
```

**Exception-handling tables (Informatica):**
1. **Error message** – success/error messages after execution.
2. **Data message** – bad data / error data, session id.
3. **Session** – session error info.
4. **Transformation** – transformation-level info.

---

## 22. Informatica & ETL

**Informatica** – American software company (founded 1993, California). Platform for integrating data from databases, applications and cloud platforms. Graphical, drag-and-map interface (source → target).

> Limitation: needs fixed structure; for non-fixed structures use UNIX scripts / stored procedures.

**ETL** = **Extract → Transform → Load** – extract from multiple sources, transform into a suitable format, load into the data warehouse.

**ETL tools:** Informatica (~23% market), DataStage, Talend, AWS Glue, Pentaho, Fivetran, Oracle, Hevo Data.

- **Source** – read / pull data
- **Target** – write / insert data
- **Domain** – maintains metadata

### Components
| Component | Purpose |
|-----------|---------|
| **Repository** | Maintains metadata & provides access to other services; server can't run without it |
| **Designer** | Create mappings showing data flow between source and target |
| **Workflow Manager** | Create workflows, sessions and tasks; maintain dependencies |
| **Workflow Monitor** | Monitor workflow execution; shows success / failure |

### Two-tier (Client–Server) Architecture
- Services: **Repository Service** and **Integration Service**.
- **Server** stores data; **Client** provides instructions.
- Used for non-complex, non-time-critical systems.
- **Advantages:** easy to maintain, quick communication, compatible.
- **Disadvantages:** poor performance, application reinstall required, needs high processing power.

1. **Repository** – stores source/target definitions, user id, name, password, execution permissions.
2. **Integration Service** – validates permissions in the repository; processes data from source to target.

### Key terms
- **Session** – instructs data transfer from source to target; runs the task.
- **Mapping** – represents data flow between source and target (field matching).
- **Load process** – moving data from staging table to base object. Integration Service runs pre-SQL → load → post-SQL (on success).
- **DTM (Data Transformation Manager)** – thread, smallest unit of work: **Read → Transform → Write**.

### Shared vs Normal Folder
| Normal Folder | Shared Folder |
|---------------|---------------|
| Private – only creator can access | Accessible by multiple users |
| Individual use and testing | Collaborative / team work |
| Limited permissions | Multi-user permissions |
| Personal, isolated dev work | Shared project work |

### Shared Drive vs Shared Folder (repository)
| Shared Drive | Shared Folder (Repo) |
|--------------|----------------------|
| Team drive, stores data | Stores repository objects |
| Used in ETL process | Used for mappings, workflows, sessions |
| Stored on network file system | Stored in repository system |
| Managed by file system | Managed by repository system |

---

## 23. Informatica Transformations

A **transformation** is a mapping object with rules defining how data is read, modified or passed to the target.

### Active vs Passive

| Active | Passive |
|--------|---------|
| Changes row count | Does not change row count |
| Changes row position | Does not change row position |
| Changes transaction boundary | Does not change transaction boundary |
| Changes flag | Does not change flag |
| Affects data flow, uses more resources | Does not affect data flow, uses fewer resources |
| e.g. Filter, Router, Rank, Sorter | e.g. Expression, Sequence Generator |

> Whether a transformation is active or passive depends on its natural behavior.

### 1) Filter
- Works like SQL `WHERE`. **Active, connected.**
- Removes unwanted records; **cannot store rejected data**; only one group of condition and one output.
- Properties: Tracing level (Normal, Terse, Verbose Initialization, Verbose Data), filter condition.
- **Performance tip:** use Filter as early as possible in the mapping.

### 2) Sorter
- Works like SQL `ORDER BY`. Sorts ascending/descending on one or more keys.
- **Active, connected** – because of the *Distinct* property (removes duplicates → changes row count).
- Creates a cache at runtime.
- Properties: Distinct, Case sensitive, Null treated low/high (default high), ports I / O / K (key).

### 3) Router
- Works like Filter but supports **multiple conditions** and **multiple output groups**.
- **Active, connected.** Rejected data goes to the **default group**.
- Properties: Tracing level, rename, make reusable.

### 4) Aggregator
- Works like `GROUP BY`; performs `AVG, SUM, MIN, MAX, COUNT` on groups.
- **Active, connected.** Ports: I, O, V (variable), Group By.
- Needs **cache** (Data cache = non-group columns, Index cache = group-by columns).
- **Performance tip:** provide **sorted input**.
- Without Group By, the whole table is one group and the **last row** value is returned.
- Aggregate functions can be nested only to two levels.
- If sorted input is checked and data isn't actually sorted by the group-by port, the mapping fails.
- Properties: Cache, Tracing level, Sorted Input, Data/Index cache size, Transaction scope.

### 5) Rank
- Assigns rank to rows; select **top or bottom N**.
- **Active, connected.** Uses **R (rank) port**; needs cache (sorting, grouping, rank index).
- Behaves like RANK (1,2,2,4,5).
- Properties: Tracing level, Rank port, Cache directory, Top/Bottom, Number of ranks, Case sensitive, Data/Index cache size.

### 6) Source Qualifier (SQ)
- **Active, connected, default** transformation for relational/flat-file sources.
- Converts source data types into Informatica-understandable types.
- Not reusable; supports all operators (`>`, `=`, `<`).
- Joining in SQ needs **homogeneous** sources (same DB, same columns/sequence).
- Properties: SQL query (override), User-defined join, Source filter, Number of sorted ports, Pre-SQL, Post-SQL, Select distinct.
- **Types:** Default (generated query) / Custom (manual complex SQL override).
- Session-level query override > mapping-level generated query.

### 7) Joiner
- **Active, connected.** Joins heterogeneous or homogeneous sources.
- Only `=` operator (use Lookup for other operators).
- Two inputs: **Master** (smaller, cached) and **Detail**.
- Number of joiners = sources − 1.
- **Performance tip:** choose the table with fewer rows as master; use sorted input.
- **Join types:**
  - **Normal** – only matching rows
  - **Master Outer** – all detail rows + matching master rows
  - **Detail Outer** – all master rows + matching detail rows
  - **Full Outer** – all matching and non-matching rows from both

### 8) Sequence Generator
- **Passive, connected.** Generates unique numbers (surrogate keys); **no input port**.
- Ports: `NEXTVAL`, `CURRVAL`.
- Properties: Tracing level, Increment by (+1), Start value, End value, Current value, Cycle, Number of cached values (default 0).
- Mapping fails if End value < number of source rows (without cycle).

### 9) Union
- Behaves like **UNION ALL** (returns duplicates too).
- **Active, connected.** All input groups need the same structure.
- Properties: Rename, Description, Reusable, Tracing level, Module/Function identifier, Runtime location, Transaction scope.

### 10) Expression
- **Passive, connected.** Row-by-row calculations; **no cache** (cannot hold data for next row).
- Ports: **I** (input), **O** (output), **V** (variable). Execution order: I → V → O.
- Functions: numeric, string (`CONCAT`), `IIF`, etc.
- Performance tip: use minimum transformations in one mapping.
- Default values: Integer = 0, String = NULL, Date = 1 Jan 1753.

### 11) Transaction Control
- Commit or rollback transactions; useful for large volumes (e.g. commit every 10000 rows).
- **Active, connected.** Row-by-row; commit generates a new file.
- Flags:
  - `TC_COMMIT_BEFORE` – commit before current row
  - `TC_COMMIT_AFTER` – commit after current row
  - `TC_ROLLBACK_BEFORE` – roll back before current row
  - `TC_ROLLBACK_AFTER` – roll back after current row
  - `TC_CONTINUE_TRANSACTION` – continue (default)

### 12) Lookup
- Behaves like **Source Qualifier + Master Outer Join** (default = left outer join).
- Can be: **Active / Passive**, **Connected / Unconnected**, **Cached / Uncached**, **Persistent / Non-persistent**, **Static / Dynamic**.
- Defaults: Passive, Cached, Left outer join.
- Used to check if a record exists – returns value if found, else `NULL`.
- **Lookup source** can be: Source, Target, Source Qualifier (pipeline lookup – better performance), or Import (database / file).
- **Active lookup** – can return multiple matches (changes row count). **Passive** – returns a single value.
- **Cache types:**
  - **Static** – cannot change during the session
  - **Dynamic** – can change during the session
  - **Persistent** – stored permanently (Named = shared, Unnamed = unshared)
- Properties: Tracing level, SQL override, Lookup table, Lookup source filter, Lookup cache enabled, Lookup policy on multiple match, Source type, Cache directory, Persistent cache name, Re-cache.

### Joiner vs Source Qualifier

| Joiner | Source Qualifier |
|--------|------------------|
| Used to join data | Retrieves data from source |
| Only `=` operator | All operators allowed |
| Can be reusable | Not reusable |
| Homogeneous + heterogeneous sources | Needs homogeneous sources, same structure & sequence |
| Cannot override query | Can override query |

### Filter vs Router

| Filter | Router |
|--------|--------|
| Single condition | Multiple conditions |
| One output group | Multiple output groups |
| Pass or discard | Pass or store |
| Drops rejected data | Rejected data kept in default group |
| Less flexible | More flexible |

---

## 24. Flat Files

| Fixed-width file | Delimited file |
|------------------|----------------|
| Continuous string, no delimiter | Fields separated by comma, dot, space, pipe, etc. |
| Record stored in a single line | Uses delimiter to separate fields |
| Properties: fixed width, no delimiter, padding, row uniformity, schema dependency | Properties: delimiter, start import at row (header skip), remove header |

**Creating a file source:** Server → Shared → Cache → SrcFiles → create file → save. Import source as *flat file*.
To add a header: use the header option / output field names.

### Indirect vs Dynamic File Loading

| Indirect File Loading | Dynamic File Loading |
|-----------------------|----------------------|
| Convert **multiple files → single target** | Convert **single source → multiple files** |
| File names come from an external **control (list) file** | File name generated automatically at runtime |
| Used on the source side (reading) | Used on the target side (writing) |
| Set up manually; less flexible | Happens automatically; more flexible |
| Batch of files listed in control file (same structure) | Creates output file for each region / date dynamically |

---

## 25. CDC, Variables & Target Load Plan

### CDC – Change Data Capture
Load only **new / changed** data and skip old data.
- Identify using **Insert timestamp** / **Update timestamp**; default value = system date.
- Uses **mapping variables** / **parameters**.

| Item | Description |
|------|-------------|
| **Transformation variable** | Normal variable used in an Expression |
| **Mapping variable** | Holds previous value in the repository; can change during the run |
| **Mapping parameter** | Value stored in a parameter file; **cannot change** during the run |

**Variable aggregation types**
1. `SetCountVariable` – total count
2. `SetMaxVariable` – maximum value
3. `SetMinVariable` – minimum value
4. `SetVariable` – assigns a value (default)

### Target Load Plan
- Defines **parent–child / dependency order** of loading multiple targets.
- Set in: *Mapping → Target Load Plan*.
- Session property: **Constraint Based Loading (CBL)** – auto-detects constraints (parent–child relationship).

---

## 26. Ways to Remove Duplicates

1. `DISTINCT` (SQL, UNION, Sorter with Distinct)
2. `GROUP BY` (with `COUNT`, `SUM`, `AVG`)
3. `ROW_NUMBER()` / `ROWNUM` with `PARTITION BY` and `ROWID`
4. `ROWNUM` with subquery
5. Transformations: **Aggregator, Sorter**
6. **Rank** transformation (keep rank = 1 / filter rank > 1)
7. Joiner and Source Qualifier
8. Filter using an Expression flag (`Y` / `N`)

---

## 27. Quick Comparison Tables

### Indirect vs Dynamic File Loading → see [Section 24](#24-flat-files)

### ROWID vs ROWNUM → see [Section 14](#14-pseudocolumns)

### Active vs Passive transformations → see [Section 23](#23-informatica-transformations)

---

## About

Notes compiled for learning **SQL, Data Warehousing and Informatica PowerCenter**.
Feel free to fork, star ⭐ and contribute.
