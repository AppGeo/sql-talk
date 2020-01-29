# Joins

Joins (or relations) are considered by many people to be so fundamental to relational databases that they named the type of database after the feature, but those people are wrong, the relational in relational databases referrers to something else, which is complex and mathematical.

Imagine we have 2 tables

table1

| key | value1 |
|-----|-------|
| 1   | a     |
| 2   | b     |
| 3   | c     |

table2

| key | value2 |
|-----|-------|
| 1   | a     |
| 2   | b     |
| 4   | d     |

## Inner Join

It returns all the rows that match and none of the rows that don't. It's the default join that you get if you don't specify what kind.

The output is as follows

| key | value1 | value2 |
|-----|--------|--------|
| 1   | a      | a      |
| 2   | b      | b      |

For this type of join (and only an this type), you can just list multiple tables in the from part of the query and then put how they match in the where clause. Note since there are 2 columns name field we have to specify which one we want in the output.

```sql
select table1.key, value1, value2 from table1, table2 where table1.key = table2.key;
```

More generally for all joins you can specify a join with an `on` clause and then you can put any number of query clauses to specify how things should match up.

```sql
select table1.key, value1, value2 from table1 inner join table2 on (table1.key = table2.key);
```

You can use a `using` clause where you specify a key that has the same name in both tables that should be used for matching, any fields listed here are deduplicated in the output (hence why we don't have to specify which 'key' we want).

```sql
select key, value1, value2 from table1 join table2 using (key);
```

We're going to use the `using` syntax for the other joins instead of the `on` but know that both work.

It returns all combinations of the tables so if the tables had been

table1

| key | value1 |
|-----|-------|
| 1   | a     |
| 1   | g     |
| 2   | b     |
| 3   | c     |

table2

| key | value2 |
|-----|-------|
| 1   | a     |
| 1   | e     |
| 2   | b     |
| 4   | d     |

the result would have been

| key | value1 | value2 |
|-----|--------|--------|
| 1   | a      | a      |
| 1   | a      | e      |
| 1   | g      | a      |
| 1   | g      | e      |
| 2   | b      | b      |

## Left Join

Left join finds all the matches (like inner join) but also returns rows that didn't match from the table you specify first in the from clause (aka the one on the left as you write it)

```sql
select key, value1, value2 from table1 left join table2 using (key);
```

| key | value1 | value2 |
|-----|--------|--------|
| 1   | a      | a      |
| 2   | b      | b      |
| 3   | c      | NULL   |

it is otherwise identical so if we had the extra values in there it would look the same except with that extra row for key 3

| key | value1 | value2 |
|-----|--------|--------|
| 1   | a      | a      |
| 1   | a      | e      |
| 1   | g      | a      |
| 1   | g      | e      |
| 2   | b      | b      |
| 3   | c      | NULL   |

## Right Join

The opposite, includes matches, plus any rows on the table you join to (aka on the right when you write it) so

```sql
select key, value1, value2 from table1 right join table2 using (key);
```

returns

| key | value1 | value2 |
|-----|--------|--------|
| 1   | a      | a      |
| 2   | b      | b      |
| 4   | NULL   | d      |

## Full Join

This does both, returning both matching rows and all non matching rows

| key | value1 | value2 |
|-----|--------|--------|
| 1   | a      | a      |
| 2   | b      | b      |
| 3   | c      | NULL   |
| 4   | NULL   | d      |

## Cross Join

This is where you don't specify a join criteria and it just returns all the combinations of rows between the 2 tables, I've never used this you might need to but your on your own for that.

## Updates

You can also use joins in an update

```sql
UPDATE
  crashes
SET
  segment_key = s._key
FROM
  segments as s
WHERE
  crashes.routeid = s.route_id
AND
  s.from_measure <= crashes.measure
AND
  crashes.measure <= s.to_measure;
```

the table for a join (or really type of query) can really be anything, including just a list of values (useful if you want to batch update some stuff)

```sql
UPDATE
  user_names
SET
  name = othertable.name
FROM
  (VALUES (232, 'ross'), (6432, 'calvin'), (86232, 'ralph'))
AS
  othertable (id,name)
WHERE
  user_names.id = othertable.id;
```

## Warning

If you find yourself doing a query, iterating through the results, and doing another query for each item in the results, you are probably doing something that could be done in a join instead, if you find yourself tempted to do that, go talk to somebody else to sanity check yourself first.
