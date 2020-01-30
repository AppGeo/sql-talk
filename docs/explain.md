# Explain

You can see how an query will be performed by putting `explain` in front of it, postgres will consult the query analyzer and tell you how it thinks it might run it

You can see how a query was run by putting `explain analyze` in front of it, postgres will run the query and then give you details about how it went

There is a tool called the postgres explain viewer https://tatiyants.com/pev/#/plans/new that gives some good visualizations to help you make sense of things

Example Output

```sql
explain analyze select "roles".* from "users" inner join "roles" on "roles"."name" = "users"."role" where lower("username") =  'email@domain.com';

Nested Loop  (cost=0.15..9.35 rows=1 width=280) (actual time=0.055..0.061 rows=1 loops=1)
  ->  Seq Scan on users  (cost=0.00..1.12 rows=1 width=32) (actual time=0.023..0.029 rows=1 loops=1)
        Filter: (lower(username) = 'email@domain.com'::text)
        Rows Removed by Filter: 7
  ->  Index Scan using roles_pkey on roles  (cost=0.15..8.17 rows=1 width=280) (actual time=0.015..0.015 rows=1 loops=1)
        Index Cond: (name = users.role)
Planning Time: 0.691 ms
Execution Time: 0.252 ms
(8 rows)
```

You'll note that we don't use an index for the lookup of the username because there are only 7 rows in the table so it's quicker to just look through them all, here is an example where doing it different ways gets different result

Bad

```sql
 explain analyze  select "school_id" from "school" where name ilike 'name';

 Seq Scan on school  (cost=0.00..38.10 rows=1 width=4) (actual time=0.169..2.205 rows=1 loops=1)
  Filter: (name ~~* 'name'::text)
  Rows Removed by Filter: 1527
Planning Time: 0.293 ms
Execution Time: 2.238 ms
(5 rows)
```

good

```sql
explain analyze  select "school_id" from "school" where lower(name) = 'name';

Index Scan using school_name_school_id_lower_idx on school  (cost=0.28..8.29 rows=1 width=4) (actual time=0.123..0.125 rows=1 loops=1)
  Index Cond: (lower(name) = 'name'::text)
Planning Time: 0.100 ms
Execution Time: 0.150 ms
(4 rows)
```
