# Misc Stuff

## truncate

If you are deleting everything in a table, instead of running `delete from TABLE` you can run `truncate TABLE`

The benefits of truncate over delete
- it is generally faster
- it doesn't require you to vacuum the table to free up that space for the database to use for other stuff

The drawbacks are
- it doesn't support where clauses so you can't use it to remove parts of the table, all or nothing
- It is not transaction safe, if you truncate and then fill a table inside of a transaction block outside things might see it empty (that being said it can be rolled back if there is an error, so you can use it inside a transaction)

## Upserts

Postgres has traditionally not had any sort of upsert but as of version 9.5 you can, the requirements is to have a unique key constraint, so if you have a table, with 2 columns key, and value, and want to make the the row with  key 1 has a value of 'a' then what you could do is

```sql
Insert into table (key, value) values (1, 'a') on conflict(key) do update set value = excluded.value;
```

There is a temp table made called 'excluded' for you to use to then update the rows

## strings/quotes

single and double quotes are not interchangable in postgres, `identifiers` such as table and column names use double quotes, while `values` like  a string you pass in are in single quotes

Double quotes are usually optional for tables and columns, but if you make a table in camelCase in knex, then to query it anywhere else requires double quotes, also there is a built in table named `user`
 in postgres so if you name your table that you have to use double quotes to access it.

Postgrs has an alternative method of single quoting with dolar signs, it's 2 dollars sings, and maybe stuff between them, so the follwing are all equivilent

```sql
'foo';
$$foo$$;
$blahblahblah$foo$blahblahblah$;
```

## Environmental variables

You can avoid specifying passwords or config in any sort of config and instead pass them in environmental variables, very handy when using app engine or kubernetes as, helpful ones are `PGHOST`, `PGDATABASE`, `PGUSER`, `PGPASSWORD` and `PGSSLMODE`

## psql

psql is the command line interface for using postgres, some good tricks

`\dt`

lists all the tables

`\d <table>` gives details about a specific table including data types, constraints and indexes

`\x auto` switches to a different display mode that is helpful when there are a lot of columns in a table

`\l` lists all the databases on a server

`\c database_name` connect to different database

`\copy { table | ( query ) } { from | to } { 'filename' } [ [ with ] ( option [, …] ) ]` can be used to import and export data, it has a slightly dif

## shp2pgsql

this is a great tool that comes with postgis that makes it very easy to to bring a shapefile into postgres

```bash
shp2pgsql -s 4326 -g geom -S thing.shp table_name | psql
```

if the shapefile is in a different projection you can get it to project by changing `-s xxxx:4326` if you don't use the -s option then it give a srs of 0 which is probably not what you want

by default it creates a table, but you can do `-d` to drop and recreate or `-a` appends

`-g` is used to specify the geometry field

-S makes it use single geometries instead of multi geometries use this if all the polygons/lines are single part, you can't use it if some are multipart

## Custom functions
You can define custom functions, I think I've done that once but I can't remember why. You definitely need them if you have code that needs to run on update or on insert (like a trigger).  [The docs](https://www.postgresql.org/docs/12/sql-createfunction.html) have some information on this.

Guido can speak to this if we have more questions.

## Computed columns

As of postgres 12 computed columns are called 'generated columns' with the information `GENERATED ALWAYS AS (<expression>) STORED`

```sql
CREATE TABLE people (
    height_cm numeric,
    height_in numeric GENERATED ALWAYS AS (height_cm / 2.54) STORED
);
```

You can get more details [in the docs](https://www.postgresql.org/docs/12/ddl-generated-columns.html)

in previous versions you'd have to do complicated stuff with triggers in order to get this result (maybe that's where I used a function before).

That being said if you only need the generated thing in order to do queries on the table, then a computed index might be a better thing for you.

## Bulk data operations

There is a 'copy' command that allows you to copy csv data into and export from a table, to bulk export you can run

```sql
copy (select * form table) to './file.csv' with csv header force quote *;
```
or if you were on a different machine then the database you could change the query to be

```sql
copy (select * form table) to STDOUT with csv header force quote *;
```

put it in a file called `file.sql` and run

```bash
psql -f file.sql > file.csv
```

(you may or may not need the `force quote *` thingy this was copied from something I had created a couple months ago that needed it).

For inserts I tend to write custom write streams using knex since we often have to do data manipulation at the same time, that's out of scope for this topic, but maybe can be covered in the STREAMS! talk.

## Limit and Offset for Paging

It might seem like a good idea to do paging by running something like


```sql
-- page 1
select * from table limit 20
-- page 2
select * from table limit 20 offset 20
-- page 10
select * from table limit 20 offset 180
```

this isn't something that postgres is able to optimize away so

```sql
select * from table limit 20 offset 180
-- equivalent to
select * form table limit 200;
-- and then throwing away the first 180 rows
```
this isn't terrible if it's really just 200, but if it was like 2000 that might be a different issue.

there are a couple ways to do this better, if you only need previous and next page, then you can do

```sql
select * from table order by primary_key limit 20
```

then remember the key from the last item on the list and when they want to go to the next page

```sql
select * from table where primary_key > ? order by primary_key limit 20
```

There are more complex arraignments if you need to be able to page to arbitrary locations and you have a non trivial amount of pages that are probably out of scope for this.

## Foreign Keys

A Foreign key constraint means that you are saying that any key a certain column in this table MUST match a key in aa column in a different table.

One nice thing you can specifying when creating a foreign key is the `on delete cascade` option which says that when the key I'm referencing is deleted, also delete me.

The key being referenced must have a `unique` or `primary key` constraint placed on them, and both of those automatically create indexes meaning the referenced key will always have an index.

## Views

Views are like with statements that have been defined globally in the database and can be treated for the purpose of selects statements to be just like tables.

Other cool things:

If a view only has one table it calls from, doesn't have any aggregates and some other things then you can actually use it to update and insert data and postgres will write it to the table that is referenced.

Materialized views are views where the output is saved to disk.  They don't automatically update when the tables they are based on are updated, but there is a command to refresh them that will do so (this is similar to creating a table based on a query, but postgres saves the query that created it)
