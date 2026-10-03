---
name: sql-from-data-model-ty
description: >-
  Writes read-only SQL Server queries from a structural data model the user
  chooses. Asks for the data-model path before writing SQL. Uses only tables,
  columns, keys, and foreign keys in that file. Use when the user asks for a
  query, SQL, a DBeaver script, or to find or check rows and a data model
  exists or will be provided.
---

# SQL from Data Model (Ty)

The user gives a **scenario** (who or what they need to find or verify). You write **read-only SQL** they can paste into DBeaver. You do not run SQL against the database.

This skill does not ship a data model. The user chooses the file, or gives its path. Table and column names come only from that file.

The file is a structural catalog: one section per database, an Identity block with `database_name`, a table inventory, and per-table keys, foreign keys, checks, indexes, and columns. That is the shape `discover-data-model-ty` writes. That skill is in this same repo. This skill does not build or refresh the file.

## Ask for the data model first

Do this before any SQL, and before assuming which database the scenario means.

Use a file immediately, without asking, only when this turn or an earlier turn in the same conversation already has one of these:

- a path the user gave
- a data-model file they attached
- a choice they already made from a list you showed

Otherwise stop. Do not write SQL in that message. Ask them to choose a data model or provide its path.

If the workspace has data-model files, list them and ask which one to use. A file counts when its name or frontmatter says it is a data model, or it has an Identity section with `database_name`. If you find none, ask for the path. Do not pick one yourself. Do not use a model from memory, another project, or a previous chat.

When they answer, read that file before writing SQL. If the path does not exist, say so and ask again.

A file may contain more than one database. Use only the section whose Identity `database_name` matches the database in the script. If the scenario does not say which database, and the file has more than one, ask which section to use.

## Non-negotiable

1. **Read the chosen data model before writing SQL.** Every table, column, type, key, and foreign key in the script must appear in that file for the database you name. Do not invent names from memory, an older chat, or another database's section.
2. **Read-only only.** `SELECT` / `WITH` / `DECLARE` for filters. No `INSERT` / `UPDATE` / `DELETE` / `MERGE` / `TRUNCATE` / `ALTER` / `DROP` / `CREATE` / `EXEC` of user data.
3. **Name the database** above every script, using `database_name` from that section's Identity block. Wrong database is the most common failure.
4. **One database per script.** Do not join across databases, even when two catalogs live in the same file.
5. **DBeaver-safe scripts.** One result set per script. Do not chain `@sql` / `sp_executesql` across batches. Prefer several short scripts over one mega-script. Do not use `off` as an alias. Keep `CASE` in the select list of a simple `SELECT`.
6. **If the model does not have the field, say so.** Do not guess a column. Offer a documented column that is actually in the model, or stop. A `sys.columns` probe is only for a column the user says exists and the model does not list.
7. **SQL Server only.** These scripts target SQL Server or Azure SQL. If the user says the target is Postgres, MySQL, Oracle, or Snowflake, stop.

## Workflow

### 1. Restate the scenario

In one or two sentences: what they want to find, which identifiers they already have, and which `database_name` you will query.

Ask only if you cannot write a useful first query without a missing identifier. Prefer a query that discovers the identifier over a clarifying question.

### 2. Map scenario → tables

Open the chosen data model. For each needed fact, write down:

| Fact | Schema | Table | Column(s) | Database |
|---|---|---|---|---|
| … | … | … | … | `database_name` |

Join on a foreign key the model documents. If there is no foreign key, join only when both columns are in the model and you say the join is inferred from matching names. Do not invent a key.

### 3. Write the queries

Default template (every script):

```sql
SET NOCOUNT ON;
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

DECLARE @Id NVARCHAR(50) = N'…'; /* only if needed; match the column type in the model */

SELECT …
FROM schema.TableName
WHERE …;
```

Rules:

- Use the schema from the inventory. Put it on every table.
- Bracket reserved names (`dbo.[User]`).
- Declare variables with a type taken from that column's `type_display`. Use `NVARCHAR` for national string types and `VARCHAR` for the others.
- Filter with `DECLARE` when the user gave an id. Do not hardcode a value they will reuse.
- Filter out stale rows only when that column is in this table's column list (`DeletedAt`, `RemovedAt`, `ValidTo`, or a status column the model actually has). Do not add those filters because another database has them.
- Split: (1) confirm the entity exists, (2) the lookup, (3) a fallback only if (2) can be empty and the model has another table that could answer it.
- Label each script with a one-line comment of what it answers and which database.

### 4. Deliver

For each script, state:

- **Database:** full `database_name` from Identity
- **What it answers**
- **How to read the result** (which column is the value they asked for)
- **What empty means** and the next script if empty

Do not wrap the SQL in a file unless the user asks. They paste into DBeaver.

When they paste results, interpret them against the same table map. Offer the next query only if the scenario is still unanswered.

## Words that are not columns

A word in the scenario ("role", "owner", "active", "manager") is a column only when the chosen model has that column, or a column the model names for that idea. If the model has several candidates, say which one you used. If it has none, say the model does not record that fact.
