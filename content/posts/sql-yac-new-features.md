+++
date = '2026-05-26T10:33:10+02:00'
draft = false
title = 'SQLYac New Features'
+++

I just added a couple of new features to [SQLYac](https://github.com/kalli/sqlyac), my sql cli tool. SQLYac brings structure to your sql files and queries, making them version controlled and shareable and enables variables for reuse and readability. It gives you the power and flexibility of the terminal and the unix philosophy while avoiding lock in with expensive and proprietary gui tools. [See my previous blog post](https://dev.karltryggvason.com/sqlyac-structure-and-tooling-for-your-sql-files/) for more details; an overview of the new features follows.

## DB annotations and the execute flag (-x)

You can now annotate your files or queries with `-- @db <name>` to easily run queries against that database. This annotation is coupled with a `connections` config of the same name in `~/.sqlyac/config.json` (personal) or `.sqlyac.json` in your project. For example, let's say you have the following `example.sql` file: 

```sql
---
-- @name GetUserOrderSummary
-- @desc Per-user order count, total spend, and average order value
-- @db local
SELECT
    u.username,
    COUNT(o.id) as order_count,
    COALESCE(SUM(o.total_amount), 0) as total_spent,
    COALESCE(AVG(o.total_amount), 0) as avg_order_value
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.username
ORDER BY total_spent DESC;
```

And the following config:

```json
{
  "connections": {
    "local":  { "command": "sqlite3 ./app.db" }
  }
}
```

Instead of having to run `sqlyac example.sql GetUserOrderSummary | sqlite3 ./app.db` you can now just write `sqlyac example.sql GetUserOrderSummary -x`:

<div style="width: fit-content; font-family: monospace; white-space: pre;background-color: #282c34;color: #ffffff; padding: 1em; border-radius: 5px;"><div style="display: inline;color: rgb(204, 102, 102);font-weight: bold;">&#10140;  </div><div style="display: inline;color: rgb(138, 190, 183);font-weight: bold;">sqlyac</div> <div style="display: inline;color: rgb(129, 162, 190);font-weight: bold;">git:(</div><div style="display: inline;color: rgb(204, 102, 102);font-weight: bold;">main</div><div style="display: inline;color: rgb(129, 162, 190);font-weight: bold;">) </div><div style="display: inline;color: rgb(240, 198, 116);font-weight: bold;">&#10007;</div> sqlyac example.sql GetUserOrderSummary -x 
username                             order_count  total_spent  avg_order_value
-----------------------------------  -----------  -----------  ---------------
charlie                              2            2400         1200.0         
bob                                  4            1079.98      269.995        
alice                                4            90.98        22.745         
diana                                2            50           25.0           
<div style="display: inline;color: rgb(181, 189, 104);font-weight: bold;">&#10140;  </div><div style="display: inline;color: rgb(138, 190, 183);font-weight: bold;">sqlyac</div> <div style="display: inline;color: rgb(129, 162, 190);font-weight: bold;">git:(</div><div style="display: inline;color: rgb(204, 102, 102);font-weight: bold;">main</div><div style="display: inline;color: rgb(129, 162, 190);font-weight: bold;">) </div><div style="display: inline;color: rgb(240, 198, 116);font-weight: bold;">&#10007;</div> </div>

The annotation can be set at the file or individual query level, if say you have specific commands for just your local or staging databases. You can override the db set in the file by passing `--db <name>` to the command.

Each connection is just a shell command that runs through `sh -c` so you can still pipe, slice and dice the results however you'd like. The old way (piping into your sql client) still works as well and is the default, but `-x` is faster and more ergonomic.

## LS and query descriptions

A new `ls` command lists the queries in a given `.sql` files along with relevant information (if any). Run `sqlyac ls example.sql` to show all queries (actually `sqlyac example.sql` will do the same thing).

The `@desc` annotation allows you to add a description to complex queries, giving you more documentation and a humanly readable way of understanding the queries. Descriptions and the db annotations are shown in the ls output:

<div style="width: fit-content; font-family: monospace; white-space: pre; background-color: #282c34; color: #ffffff; padding: 1em; border-radius: 5px; overflow: visible;" ><div style="color: rgb(181, 189, 104); font-weight: bold;">➜  </div><div style="display: inline; color: rgb(138, 190, 183); font-weight: bold;" >sqlyac</div> <div style="display: inline; color: rgb(129, 162, 190);font-weight: bold;">git:(</div><div style="display: inline; color: rgb(204, 102, 102);font-weight: bold;">main</div><div style="display: inline; color: rgb(129, 162, 190);font-weight: bold;">) </div><div style="display: inline; color: rgb(240, 198, 116);font-weight: bold;">✗</div> sqlyac example.sql    
NAME                 DESC
CreateUsersTable     Create the users table with id, username, email, and active flag
CreateOrdersTable    Create the orders table with a foreign key to users
InsertSampleUsers    Seed the users table with four sample rows
InsertSampleOrders   Seed the orders table with a mix of completed, pending, and cancelled orders
GetAllUsers          List every user, oldest first
GetActiveUsers       List active users sorted by username
GetLargeOrders       Orders over $100, joined with the owning user, highest first
GetUserOrderSummary  Per-user order count, total spend, and average order value
GetRecentOrders      Orders placed in the last 7 days, newest first
CountOrdersByStatus  Count orders grouped by status, largest bucket first
CleanupTestData      Drop the orders and users tables — destructive, used to reset the example
QueryWithVariables   Demo of @variable interpolation — selects orders for a given user/status with a row limit

12 queries
<div style="display: inline; color: rgb(181, 189, 104);font-weight: bold;">➜  </div><div style="display: inline; color: rgb(138, 190, 183);font-weight: bold;">sqlyac</div> <div style="display: inline; color: rgb(129, 162, 190);font-weight: bold;">git:(</div><div style="display: inline; color: rgb(204, 102, 102);font-weight: bold;">main</div><div style="display: inline; color: rgb(129, 162, 190);font-weight: bold;">) </div><div style="display: inline; color: rgb(240, 198, 116);font-weight: bold;">✗</div>                        </div>

## Autocomplete and search

Pressing tab while at the file or query part of a command will now offer autocomplete matches to your currently typed input:

<div style="padding: 1em; border-radius: 5px; background-color: #282c34; width: fit-content; font-family: monospace; white-space: pre;background-color: #282c34;color: #ffffff;"><div style="display: inline;color: rgb(181, 189, 104);font-weight: bold;">&#10140;  </div><div style="display: inline;color: rgb(138, 190, 183);font-weight: bold;">sqlyac</div> <div style="display: inline;color: rgb(129, 162, 190);font-weight: bold;">git:(</div><div style="display: inline;color: rgb(204, 102, 102);font-weight: bold;">main</div><div style="display: inline;color: rgb(129, 162, 190);font-weight: bold;">) </div><div style="display: inline;color: rgb(240, 198, 116);font-weight: bold;">&#10007;</div> sqlyac example.sql Get
GetActiveUsers       GetLargeOrders       GetUserOrderSummary                     
GetAllUsers          GetRecentOrders                                            </div>

The search command is complementary — it matches query names, descriptions, and SQL bodies. For example: `sqlyac search example.sql users`.

Note that the autocomplete [feature requires installation](https://github.com/kalli/sqlyac#shell-completion) for your shell.

## Implementation details 

Previously sqlyac was pretty much vanilla/stdlib go. But with these new features it made sense to introduce the [Cobra cli framework](https://cobra.dev). This brought some structure to the project and basically gave me the autocomplete feature for free.

Let me know if you find SQLYac helpful, issues, PRs and contributions welcome and encouraged!
