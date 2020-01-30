# Transactions

A transaction is a set of statements of any sort (i.e. select, insert, delete, update) that either all succeed or all fail and see a consistent view of the database.

So for instance if you have multiple inserts to do to various tables, and if after you do some of them you get an error, you can `rollback` the transaction and prevent it from being left in a halfway state.

Other queries only see the end result of the query, if you do 2 different updates, other queries only see the results of both or none, never just 1.

Statements inside the transaction only see the state of the database as it was when the transaction started (so if you do 2 queries inside the block, data won't get updated in one but not the other)

Knex syntax is super simple

```js
const trx = await db.transaction();
```
then use trx instead of db for all your database needs

```js
await trx('table').select/insert/whatever
```

wrap the whole thing in a try catch and at the end do a

```js
await trx.commit();
```

and in the catch do an

```js
await db.rollback();
```

```js
const trx = await db.transaction();
try {
  const stuff = await trx('table').insert(inputs).returning('something');
  const furtherStuff =  await trx('otherTable').insert(stuff).returning('something');
  await trx.commit();
  return furtherStuff;
} catch(error) {
  await trx.rollback();
  throw error;
}
```

Transactions add overhead to the database so you don't necessarily want to use them if you don't need them. So if in that example above, if it was ok if data go into `table` but not into `otherTable` then maybe don't use a transactions, but generally if your inserting a bunch of stuff into different tables all at the same time you probably want one.

For transactions around doing a lot of reads and wanting them to be consistent ideally you should be able to get all the data you need for a request in a single query and making over a 1000 calls to a database for a single request should be considered bad form, but sometimes you might need to make a few separate selects and combine them server side in which case you may want to put it in a transaction block to make sure it doesn't get changed out from under you.

# deferable

by default foreign key and unique constraints are checked after every
You can set them to be 'deferable' with

```sql
SET CONSTRAINTS ALL Deferred;
```

inside a transaction and it will only check them at the end of the transaction block. This is handy if you have lots of complex interlocking constraints.

you can also set the constraint to be `DEFERRABLE INITIALLY DEFERRED` when you define it to make this the default behavior.

this is not supported by knex, but somebody did [write a script](https://gist.github.com/scf4/03780af508218200a590959d8258f61c) you can use to automatically set all the constraints to be deferrable.
