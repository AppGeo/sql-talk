# Vacuuming

One way that postgres is able to do these fancy types of transactions is by saving multiple versions of a row, so that when you update a field, what actually happens is that a new version of the row with the new value is inserted and then a pointer is switched to point to the new version instead of the old version, but the old version isn't deleted (the technical term for this is Multi Version Concurrency Control or MVCC)

These old versions of rows (and old entries in indexes) need to be removed, running VACUUM finds these dead rows and removes them making the disc space available to the database (But does not free up the space for other programs)

VACUUM FULL compacts the tables and frees space on the hard drive, but it involves copying the table so it requires there to be extra disc space and it requires an exclusive lock on the table

Ideally vacuum runs periodically in the background and frees up space that is then reused by the database before being reused again

That being said sometimes if you make big deletes or updates it can make sense to manually run vacuum to make sure the database doesn't grow to big

You can also run vacuum analyze to update the query planner stats on a table, usefully if you are seeing wonky stuff in your explains

Syntax is

```sql
vacuum; -- normal vacuum on all tables
vacuum TABLE_NAME; -- normal vacuum on one table

vacuum full;
vacuum full TABLE_NAME;
-- full vacuums on either all tables or one table

vacuum analyze;
vacuum analyze TABLE_NAME;
vacuum (analyze, full) TABLE_NAME;
-- run an analaze, can be combined with full

vacuum verbose;
vacuum verbose TABLE_NAME;
vacuum (analyze, verbose) TABLE_NAME;
vacuum (full, verbose) TABLE_NAME;
vacuum (analyze, full, verbose) TABLE_NAME;
-- give extra logging, can be combined
```

And remember, there is an autovacuum process that is turned on by default so if you're just doing regular stuff you shouldn't need to touch this.
