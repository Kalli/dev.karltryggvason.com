+++
date = '2025-10-13T10:11:17+02:00'
title = 'SQLYac - structure and tooling for your sql files'
+++

[SQLYac](http://github.com/kalli/sqlyac) is a tool I built to help me manage and structure sql files and run the queries within them. For many projects I've worked on I find it helpful to maintain a list of frequently used sql queries (typically in `queries.sql` or the like), for debugging, running administrative tasks, or testing purposes. I find this improves developer experience and helps document the structure of a projects database.

Here's a quick overview of SQLYac, from the ReadMe:

> SQLYac lets you write multiple sql queries in a single file and execute them individually from the command line. Write your queries in an organized file, then pipe specific ones to your database tools of choice.

## Usage

Say for example you have the following `queries.sql` file:

```sql
---
-- @name GetActiveUsers
SELECT user_id, username, last_login
FROM Users
WHERE active = 1
ORDER BY last_login DESC;
---
```

Here's some examples of how you could use the SQLYac with that file:

```bash
# list all available queries
sqlyac queries.sql

# run a specific query
sqlyac queries.sql GetActiveUsers | mysql -u user -p database

# with flags (same thing)
sqlyac --file example.sql --name GetActiveUsers | sqlite3 db.sqlite
```

## File format and syntax

Your `.sql` files should contain queries separated by three dashes (`---`) and annotated with with `@name`. SQLYac also supports variables you might want to reuse accross queries. You can define variables using the sql syntax `SET @variable_name="value"` and then use them in queries like so `@variable_name`.

## Avoid footguns

If you want to be careful (for example in production databases) you can set a confirmation prompt for queries. Using a config file: `~/.sqlyac/config.json` with the following settings:

> * `confirm` - Ask for confirmation on all queries.
> * `confirm_schema_changes` - Ask for confirmation on any queries that change the database schema (i.e. `drop table`, `alter table` etc).
> * `confirm_updates` boolean - Ask for confirmation on any queries that create, update or delete rows.

This will print the queries out and ask for confirmation before running them against the database.

## Future functionality

I want to keep this tool fairly basic, it is meant to be piped into database tools and then onwards into whatever command line shenanigans you want. I don't want to start supporting different db drivers or managing connections.

Still, I've thought maybe some sort of optional frontmatter support for connections and protocols would be neat. If you could define environments and connections that would let the tool run queries against different databases (for instance `dev`, `staging` or `production`) by making command line calls and then piping to stdout.

Some sort environment support would be good as well. You might have a different set of variables for your local environment (based on your seed data) and production for instance or for different scenarios you want to debug.

It would also be good to have some way to test the queries, to verify that you don't get drift between your sql files and the actual schema of your database.

Maybe some ai related functionality would be neat as well. I haven't fully wrapped my head around mcp servers, but think this tool and a `.sql` file could perhaps provide an interface for ai agents to understand a projects database and make queries to it.

In any case feature requests, PRs, feedback or bug reports welcome on the [Github repo](http://github.com/kalli/sqlyac).

## Shouts to HTTPYac

I based this workflow on `.http` files that are often used to manage http requests for a service or api. In fact the name, SQLYac, is a reference to what is my favourite tool in that space, [HTTPYac](https://httpyac.github.io), and SQLYac uses similar syntax and ergonomics. Although reflecting on it, the "*yac*" accronym which I believe stands for *"yet another client"* doesn't really work for SQLYac, which explicitly isn't a client. Naming things is hard.
