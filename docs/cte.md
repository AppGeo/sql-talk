# Common Table Expressions (CTEs aka With statements)

Anyone looking for information on concussions is in the wrong room, I recommend you watch that Aaron Hernandez documentary.

As hinted earlier anyplace you can specify a table by name, you can also specify an expression, meaning you can do something like

```sql
Select * from (select something from table where foo = 'bar') as foo;
```

Note you must name the expression inside the parentheses.

You can do this various places so another example might be

```sql
select * from table where id in (select key from other_table where foo = 'bar');
```

you can pull these things out in into a with statement

```sql
with foo as (
  select something from table where foo = 'bar'
)
Select * from foo;
```

the other example isn't as good

```sql
with foo as (
  select key from other_table where foo = 'bar'
)
select * from table where id in (select key from foo);
```

but could be if there was a lot of complexity inside that function, you can also use these for joins and really any place you'd be able to put a table.

You can also have multiple comma separated ones that reference each other.

```sql
With foo as (
  select something from table
),
bar as (
  select something from foo
)
select blah from bar;
```

As of version 12 of postgres these syntaxes are basically equivalent but in older versions of postgres using `with` could be slower then doing it inline because when it was inline the query analyze could see if the results were going to be filtered and do that ahead of time, so

```sql
select * from (select * from table) as foo where foo.blah = 'baz';
```
could be transformed into
```sql
select * from (select * from table where blah = 'baz') as foo;
```

but before version 12

```sql
with foo as (
  select * from table
)
select * from foo where foo.blah = 'baz';
```

would not get that optimization.

By default in 12, if a with statement is use only once it'll just get folded in, though you can tweak this, [the docs](https://www.postgresql.org/docs/current/queries-with.html) are very good and give examples of when you would and wouldn't want this.

## Knex support
```js
knex.with('name', knex.raw('text')).select().from('name');
// or

var query = knex.with('name',(qb) => {
  qb.select('*').from('books').where('author', 'Test')
})).select().from('name');
```

## Order of execution

You don't care

SQL is declarative, you declare what the results you want and the query optimizer goes about figuring out the best way to do it.  If you use explain to view how a query was actually executed it's sometimes very different from how it was written.


## Query planner

The query planner that decides how to run the query is very smart, except when it's dumb.  If you are doing a with statement where you select from a large table and latter join it to a table 

i.e. 

```sql
with something_big as (
  select id,  json_agg(big_table) AS json from big_table
)
select small_table.*, something_big.json from
small_table, something_big
where small_table.some_id = something_big.id
and small_table.id = 4;
```

You'd think the analyzer would be smart enough to only choose the rows in  big_table that match but you might have to do 

```sql
with something_big as (
  select id,  json_agg(big_table) AS json from big_table where id in (
    select some_id from small_table where id = 4
  )
)
select small_table.*, something_big.json from
small_table, something_big
where small_table.some_id = something_big.id
and small_table.id = 4;
```

despite to reads from small_table this can be WAYYYY faster.