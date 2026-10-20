# Data model skills

Skills for SQL Server and Azure SQL. One learns the shape of a database. `/sql-from-data-model` is the chooser for a question. `/sql-from-data-model-manual` is the manual path on its own.

dbvr uses the `microsoft` driver and a SQL username and password. These skills do not use Azure AD. They do not support other database engines.

This repo does not include a data model. The default location is the `model_path` from `paths`, inside the settings folder, which is outside this repo. You can choose another path. Keep the data model outside this repo and do not commit it. Do not commit passwords or connection strings. Passwords are typed in the sign-in windows and stored only in the private `.env` file.

## How they work together

Every question needs a data model: a map of the database's tables, columns, and keys. Option A builds it during first-time setup. `/discover-data-model` builds it by hand, one catalog script at a time. Option B uses a model you already have.

```mermaid
flowchart TD
  start["/sql-from-data-model"] --> menu{"Option A or B?"}

  menu -->|"A: Automatic"| done{"Setup done?"}
  done -->|No| setup["First-time setup<br/>1. Check dbvr and that each database can be reached<br/>2. Sign in through a window, never the chat<br/>3. Prove the login can only read data<br/>4. Build the data model"]
  setup --> ask
  done -->|Yes| ask["Ask a question in plain English"]
  ask --> write["The skill writes a SELECT from the data model"]
  write --> run["The script refuses anything but SELECT and WITH,<br/>then runs the query through dbvr"]
  run --> answer["Answer in the chat:<br/>the exact table, the row count, and the query"]

  menu -->|"B: Manual"| model[("Data model file")]
  discover["/discover-data-model<br/>7 catalog scripts you run yourself"] --> model
  model --> writeb["The skill writes a SELECT"]
  writeb --> dbeaver["You run it in DBeaver<br/>and paste the result back"]
```

If dbvr is not available, Option A can build the data model from your code repos instead. It then writes the query but does not run it.

## Prerequisites

- Python 3. The standard library is enough. Try `python3`, then `py -3`, then `python`.
- dbvr 26.2 or newer for Option A. The skill does not install it. Releases: https://github.com/dbeaver/dbvr/releases/latest
- Network or VPN access to the database ports.
- Your IP allowed in the Azure SQL firewall, when the server is Azure SQL.

Option B needs a data model file and DBeaver. It does not need dbvr.

## Install

Clone this repo, or use **Code → Download ZIP** on GitHub and unzip it.

```text
git clone https://github.com/Tytrana/data-model-skills-ty.git
```

**Cursor.** Copy these three folders into the user skills folder:

- Mac and Linux: `~/.cursor/skills/`
- Windows: `%USERPROFILE%\.cursor\skills\`

**Claude Code.** Copy the same three folders into:

- Mac and Linux: `~/.claude/skills/`
- Windows: `%USERPROFILE%\.claude\skills\`

**Codex, Copilot, and other agents.** Open this repo and follow `AGENTS.md`.

The three folders are:

- `.cursor/skills/discover-data-model`
- `.cursor/skills/sql-from-data-model`
- `.cursor/skills/sql-from-data-model-manual`

The dbvr script is `.cursor/skills/sql-from-data-model/scripts/sql_login.py`. Copying that skill folder copies the script. `.cursor/skills/sql-from-data-model/scripts/databases.example.json` is a placeholder list with no passwords. Copy it, replace the placeholders, and pass that file to `init-databases`. Do not commit the copy if it contains real server names.

Opening this repo in Cursor also finds the skills under `.cursor/skills/`.

## Where settings live

The script prints the paths. Run:

```text
python3 .cursor/skills/sql-from-data-model/scripts/sql_login.py paths
```

- Mac and Linux: `~/.config/sql-from-data-model/`
- Windows: `%APPDATA%\sql-from-data-model\`

That folder holds:

- `setup.json` — source, dbvr path, model path, repo links, an optional notes path, and whether read access was confirmed. No passwords.
- `databases.json` — servers and database names. No passwords.
- `.env` — username and password per database.
- `dbvr-workspace/` — dbvr’s own workspace. The skill does not use DBeaver’s folders.

`setup.json` looks like this. `repo_links` is one link per database label. `notes_path` may be empty.

```json
{
  "source": "",
  "dbvr_path": "",
  "model_path": "",
  "repo_links": {},
  "notes_path": "",
  "read_access_confirmed": false
}
```

On Mac and Linux the script sets the folder to mode `0700` and `.env` to mode `0600`. On Windows it relies on the user profile being private. Windows does not get those Unix modes.

## The chooser

In chat, type `/sql-from-data-model`.

The skill shows Option A, then Option B. Read both. Reply with A or B. It does not look anything up until you choose.

If that same message already says Option A or Option B, it skips the menu and uses that option.

You can also type `/sql-from-data-model-manual`. That starts the manual path directly.

### Option A

dbvr is the primary way, on Mac or Windows. You need dbvr 26.2 or newer, the database ports reachable, and your IP allowed by the database firewall. There is no built-in database list. You enter each database (environment, label, server, database name, and port), or you point at a JSON file shaped like `.cursor/skills/sql-from-data-model/scripts/databases.example.json`.

Setup opens one sign-in window per environment. Each database has a username and a password. Paste with ⌘V on Mac and Ctrl+V on Windows. The login is saved only after every database in that run shows reader 1, writer 0, owner 0. That check covers database roles, not permissions granted directly to the login.

An `.env` already saved with `SQL_USER` and `SQL_PASSWORD_<KEY>` keeps working. `SQL_USER_<KEY>` is used when it exists. When it does not, `SQL_USER` is the username. A complete saved login does not open the sign-in windows again.

Setup then builds the data model, one database at a time. After that, a question runs on the database and the answer comes back in the chat.

Queries use `SELECT` and `WITH` only. Every statement must start with `SELECT` or `WITH`. The only `SET` lines allowed are `SET NOCOUNT ON/OFF` and `SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED`, which the catalog scripts use. Put values directly in the query, for example `WHERE CustomerId = N'C-1001'`. Do not use `DECLARE` or `@variables`. dbvr runs each statement separately, so a declared variable is lost (SQL Error 137, verified 2026-10-16). The repo source writes queries the same way.

The skill cannot connect to DBeaver, and it does not read DBeaver’s saved passwords.

The repos are a secondary choice if dbvr is not available. You provide one repo link per database label, and read access. The skill reads migrations and entity models. It writes a query from the code. It does not run the query, because the code has no rows. This setup does not ask for passwords.

### Answers

An answer copies the script’s table exactly as printed, including the header, the separator, and every row, then states `<n> rows returned.` from the script’s `ROWS` line. The opening sentence quotes only names, numbers, dates, IDs, and statuses that appear in that table. A column is explained only when the data model or the user's notes file documents it. The notes file is optional. Its path is `notes_path` in `setup.json`. The blank template is `.cursor/skills/sql-from-data-model-manual/notes.template.md`.

### Option B — manual

The manual skill is `sql-from-data-model-manual`. Open it with `/sql-from-data-model-manual`, or pick Option B from the chooser.

The skill reads a saved map of tables and columns, writes the question, and you run that question yourself in DBeaver.

You need the data model file, and DBeaver already open on the database. The skill writes the SQL. You paste it into DBeaver, run it, and paste the result back.

The model file is `model_path` in `setup.json` when that file exists. Otherwise the skill asks which file to use.

The query uses only tables and columns in that file. Each script is for one database. Column notes come from the data model or from `notes_path`. If `notes_path` is empty, a column that the data model does not explain is reported as `Meaning not documented.`

## Discover the database

In chat, type `/discover-data-model`.

The skill gives you one SQL script. Run it in your database tool and paste the result back. It then gives you the next script. There are 7 scripts. They only read the list of tables, columns, and keys. They do not change data.

After the last result, the skill writes a markdown file. That file is the data model. Save it outside this repo. Do not commit it.

Do one database at a time. If you have two databases, finish the first before starting the second.

Option A, when it uses dbvr, runs those same catalog scripts. It does not change them, and it does not replace `/discover-data-model`.

## What has been verified

Verified on Mac with dbvr 26.2.2 against Azure SQL: login, the read-only proof (reader 1, writer 0, owner 0), SELECT, JOIN, GROUP BY, an empty result, and an emoji value.

- The Mac sign-in window: an NSAlert with an Edit menu, so ⌘V paste works.
- The read-only role proof returned reader 1, writer 0, owner 0.
- This query style works: `dbvr -data=<workspace> sql --driver=microsoft --host= --port=1433 --database= --user= --password= --disable-status [-l=100000] [-format=csv -out=<file>] "<query>"`
- `-l` and `-format=csv -out=` worked on a large result.

Also verified on Mac through `.cursor/skills/sql-from-data-model/scripts/sql_login.py run`:

- `run` works. That covers `-in=<file>` and `-l=100000`.
- `GROUP BY` and `SELECT … JOIN` returned rows.
- The same lookup with the value written in the query returned rows.
- An empty result. dbvr printed a header and a separator, and no data row. The script then printed `ROWS 0`. The shape looks like this:

```
|CustomerId|
|----------|
```

`run` sends the SQL file with `-in=<file>` and `-l=100000`. The read-only proof is still one trailing `SELECT`. After the table, `run` prints `ROWS <n>`.

## Known limits

`DECLARE` does not work. dbvr runs each statement separately, so a variable declared in one statement is missing in the next (SQL Error 137, verified 2026-10-16). Put the value directly in the query. The script refuses `DECLARE`.

## What has not been verified

- Whether `SET NOCOUNT ON` and `SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED` carry over to the next statement. They are harmless either way. The catalog scripts stay unchanged.
- dbvr’s output encoding on Windows. The script tries UTF-8 first, then the Windows code page.
- The PowerShell sign-in window.
- Building the data model through dbvr.
- These Windows install paths, which are guesses: `%ProgramFiles%\dbvr\dbvr.exe` and `%LOCALAPPDATA%\Programs\dbvr\dbvr.exe`. Windows needs one real test of setup and one query.
- The Linux sign-in window.
- The new field list: one username and one password per database. The Mac window mechanism was verified with a shorter field list.
- The role proof covers database roles only, not permissions granted directly to the login.

## Verify on a new machine

Run this once on a real Mac or Windows machine, with a person at the computer. On Mac the Python command is `python3`. On Windows it is `py -3`. The script is `.cursor/skills/sql-from-data-model/scripts/sql_login.py`.

1. Offline tests, from this repo:

```text
python3 -m unittest discover -s tests -v
py -3 -m unittest discover -s tests -v
```

Every test passes.

2. `paths` shows this machine’s settings folder. Mac and Linux: `~/.config/sql-from-data-model/`. Windows: `%APPDATA%\sql-from-data-model\`.

```text
python3 .cursor/skills/sql-from-data-model/scripts/sql_login.py paths
py -3 .cursor/skills/sql-from-data-model/scripts/sql_login.py paths
```

3. `check` prints `OK dbvr` and a version of 26.2 or newer. If it prints `STOP dbvr not found`, run it again with `--dbvr-path` set to the dbvr program.

4. Copy `.cursor/skills/sql-from-data-model/scripts/databases.example.json`. Replace the placeholders with the servers and database names you use, and save that file outside this repo. Then load it and run `check` again. Every server prints `OK port`.

```text
python3 .cursor/skills/sql-from-data-model/scripts/sql_login.py init-databases <file>
py -3 .cursor/skills/sql-from-data-model/scripts/sql_login.py init-databases <file>
```

5. `setup`, with network access and someone at the computer:
   - The “Sign in to <environment>” window opens in front. Each database has a username field and a password field.
   - Paste works: ⌘V on Mac, Ctrl+V on Windows. Passwords stay hidden.
   - Cancel saves nothing.
   - After the last window, each database prints `OK … read-only`, then `SAVED`.

6. Write a short `SELECT` from a table in one of your databases to a file and run it. `<database>` is a database name from your list. A table prints.

```text
python3 .cursor/skills/sql-from-data-model/scripts/sql_login.py run <database> <file>
py -3 .cursor/skills/sql-from-data-model/scripts/sql_login.py run <database> <file>
```

7. Write `SELECT N'📣 test' AS Sample;` to a file and run it on one database from your list. The emoji prints without an error. This is the Windows text check.

8. Run that same query with `--csv <path>`. Open the file. The emoji is intact.

9. Through the agent, build the data model for one database. The progress says “step N of 7”, and that database’s section is written. The file stays outside this repo.

10. Write down the results, including the real dbvr install path on Windows, and update the verified lists in this file, `AGENTS.md`, and `.cursor/skills/sql-from-data-model/SKILL.md`.
