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

## psql

psql is the command line interface for using postgres, some good tricks

`\dt`

lists all the tables

`\d <table>` gives details about a specific table including data types, constraints and indexes

`\x auto` switches to a different display mode that is helpful when there are a lot of columns in a table

`\l` lists all the databases on a server

`\c database_name` connect to different database

## shp2pgsql

this is a great tool that comes with postgis that makes it very easy to to bring a shapefile into postgres

```bash
shp2pgsql -s 4326 -g geom -S thing.shp table_name | psql
```

if the shapefile is in a different projection you can get it to project by changing `-s xxxx:4326` if you don't use the -s option then it give a srs of 0 which is probably not what you want

by default it creates a table, but you can do `-d` to drop and recreate or `-a` appends

`-g` is used to specify the geometry field

-S makes it use single geometries instead of multi geometries use this if all the polygons/lines are single part, you can't use it if some are multipart
