# Data model skills

Two Cursor skills for SQL Server. One learns the shape of a database. The other writes lookup queries from that shape.

This repo does not include a data model. Keep that file somewhere else. Do not commit it here.

## Install

1. Clone this repo.
2. Open the folder in Cursor.

Cursor finds the skills because they live in `.cursor/skills/`.

To use them in other projects, copy these two folders into `~/.cursor/skills/`:

- `.cursor/skills/discover-data-model-ty`
- `.cursor/skills/sql-from-data-model-ty`

## How they work together

First you build a data model. Then you point the query skill at that file.

```
database  →  /discover-data-model-ty  →  data model file (you keep this)
                                              ↓
                         /sql-from-data-model-ty  →  SELECT query you paste and run
```

## 1. Discover the database

In chat, type `/discover-data-model-ty`.

The skill gives you one SQL script. Run it in your database tool and paste the result back. It then gives you the next script. There are 7 scripts. They only read the list of tables, columns, and keys. They do not change data.

After the last result, the skill writes a markdown file. That file is the data model. Save it outside this repo.

Do one database at a time. If you have two databases, finish the first file before starting the second.

## 2. Write a query

In chat, type `/sql-from-data-model-ty` and say what you want to find.

The skill asks you to choose a data model or give the path to the file from step 1. It reads that file and writes a `SELECT` query. Paste the query into your database tool and run it.

The query uses only tables and columns that are in the file you gave it. Each query is for one database.
