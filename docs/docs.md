# How To Read Postgres Docs

postgres docs are found online at postgresql.org, specifically at

https://www.postgresql.org/docs/current/index.html

The first thing to notice on any page is the versions in the upper left, make sure to pick the one you want (when in doubt pick current), more current version tend to have more examples, though if your app is stuck on an old version you may need to pick an older version of the docs so you don't accidentally use features not supported in your version.

Google has a nasty habit of landing you on very old versions of the docs.

The part of the docs I find most helpful is the [sql section](https://www.postgresql.org/docs/current/sql.html) and the [reference section](https://www.postgresql.org/docs/current/reference.html)

The sql section some very useful parts

# [SQL section](https://www.postgresql.org/docs/current/sql.html)

## [Data types (chapter 8)](https://www.postgresql.org/docs/current/datatype.html)

This gives detailed information on different data types, if you need to use a type you're not familiar with (like a date, or a range) start here.

Note: the geometry types here are not compatable with PostGIS.

## [Functions (chapter 9)](https://www.postgresql.org/docs/current/functions.html)

This gives an exhaustive list of all the various postgres operations and functions that apply to different data types.  This is laid out similarly to the previous section and pages in chapter 8 usually have a link to the relevant pages in chapter 9 for functions that apply to the data type.


## [Indexes (chapter 11)](https://www.postgresql.org/docs/current/indexes.html)

This chapter gives very detailed information about how indexes in postgres work.

## [Table Expressions (7.2)](https://www.postgresql.org/docs/current/queries-table-expressions.html)

This is the reference I used for the information about joins in this presentation, it's very good.

# [Reference Section](https://www.postgresql.org/docs/current/reference.html)

This section has in depth descriptions WITH EXAMPLES for all sql keywords and commands.  Forget the exact syntax to insert data, [look it up there](https://www.postgresql.org/docs/current/sql-insert.html) wanna create a table from a select statement [look up Select Into](https://www.postgresql.org/docs/current/sql-selectinto.html), forget the correct syntax for creating an index [look there](https://www.postgresql.org/docs/current/sql-createindex.html).


# PostGIS

The key page is [here](https://postgis.net/docs/manual-2.5/reference.html) and lists all the functions you might ever need divided up by type.  Some of them require epsg codes, https://epsg.io is a great place to look them up, but 4326 is usually the one you want.
