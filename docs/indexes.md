# Indexes

## how/when they are used

If you have an index on a field, then any basic query that uses the field (equals, less then etc) will use it if the query analyzer things it will eliminate enough rows to be worth it

Indexes don't actually include the row itself so the db has to go look up all the rows that it finds match from the index, this is slower if it has to go a lot of the table

I.E. find me all that dates that aren't on thursdays is going to return 6/7ths of the table, so it will be faster to just go through the table and skip any rows that are thursday

But give me all the events on thursday only returns 1/7th of the table so it would be faster to use the index to find them

### like

Indexes work with `like` as long as it is 'anchored' so `foo like 'blah%'` would use the index to find all the things in column foo that begin with blah, but `foo like '%blah'` would not be able to find all the things in foo that end with blah.

But ilike or case insensitive like doesn't use the index since the index is case sensitive.

## When not to use

Never

Seriously probably never since the DB is smart enough to not use an index if it's not going to be faster

Ok so they do take up disk space so if disk space was an issue you'd maybe want to think twice there, but since disk space is so cheap that would probably only ever be an issue on one of those projects where the client wants to self host

Indexes can make inserts and updates slower so if you have a lot of indexes (and crazy computed ones) it could slow things down if your making lots of edits and inserts (This also applies to triggers)

## Types

### Btree

A regular index, god for most things

make with sql

```sql
create index IDX_NAME on TABLE (COLUMN);
```

or with knex

```js
// when defining the field
table.type('name').index();

// or later
table.index('name');
```

Knex defaults to `TABLENAME_COLLUMNNAME_INDEX` for index names that or `TABLENAME_COLLUMNNAME_IDX` are pretty good default names for indexes


removing the index

```sql
drop index IDX_NAME;
```

```js
table.dropIndex('name');
```
### GIST

A GIST index can be used for GIS qureries, this is what PostGIS uses, there are more complex ways of creating these indexes you do not need to work about

```sql
create index IDX_NAME on TABLE using GIST (COLUMN);
```

```js
table.type('name').index( 'index_name', 'gist');
// or
table.index('name', 'index_name', 'gist');
```

drop GIST indexes

```sql
-- same as a btree
drop index IDX_NAME;
```

```js
table.dropIndex('anything here', 'index_name');
```

it has to be on a geometry or geography field

### Expression Indexes

Indexes do not have to be on simply a column, they can also be on an expression.

So for instance, the way you can do a case insensitive match is

```sql
create index IDX_NAME on TABLE (lower(COLUMN));
```

or

```js
table.index(knex.raw('lower(name)'), 'index_name');
```

(remove the same as a GIST index)

then do a query in knex like

```js
.whereRaw('lower(name) = ?', [value.toLowerCase()]);
```

This can be on really any expression and as long as the exact expression is being queried against (and it makes sense to use an index) it will use that, so pre buffered geometries, simplified geometries, fields concatenated together, anything you can imagine.

### Multi Column Indexes

By default postgres can only use a single index per query (there are exceptions for parallel joins). But you can use a multi column index for queries that hit multiple columns from left to right
.

so if you have the index

```sql
create index IDX_NAME on TABLE (c1, c2, c3);
```

and the following queries

```sql
select * from table where c1 = 'something';
-- uses index
Select * from table where c1 = 'something' and c2 = 'somethingelse';
-- uses index
Select * from table where c1 = 'something' and c2 = 'somethingelse' and c3 = 'further thing';
-- uses index
Select * from table where c2 = 'somethingelse';
-- DOES NOT USE INDEX
Select * from table where c1 = 'something' and c3 = 'further thing';
-- uses index ONLY FOR c1
```

# Index only scans

By default after finding rows that fit your query it then has to look up the rows but with an index only scan you can include the other column in the index such as the following would be sped up

```sql
select a from table where b='thing';
```

to create in Version 10 or older

```sql
create index IDX_NAME on TABLE (a, b);
-- this takes advantage of multi column indexes
```

Version 11+

```sql
create index IDX_NAME on TABLE (a) INCLUDE (b);
-- you can include multiple columns in the include
```
