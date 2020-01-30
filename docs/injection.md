# SQL Injection

what are the difference between the following routes

```js
get('/path/:id', async (req, res) => {
  const result = db('table').whereRaw(`id = '${req.params.id}'`);
  res.json(result);
});
```

and

```js
get('/path/:id', async (req, res) => {
  const result = db('table').whereRaw(`id = ?`, [req.params.id]);
  res.json(result);
});
```

easy in the first one, somebody could pass in something like

```
'; drop table users; select * from passwords where '1' = '1
```

while in the 2nd one they can't

if you don't use methods that have raw in the name, you don't have to worry about sql injections, if you do use raw, then make sure you use parameters.

In a raw query a `?` a value and `??` represents an identifier, i.e.

```js
knex('users').where(knex.raw('?? = ?', ['user.name', 1]));
```

there is also a syntax for naming it using `:name` for values and `:name:` for identifier

```js
knex('users')
  .where(knex.raw(':name: = :thisGuy or :name: = :otherGuy', {
    name: 'users.name',
    thisGuy: 'Bob',
    otherGuy: 'Jay'
  }))
```

identifiers can't be `undefined`.


What you really want to avoid is big

```js
db.raw(`
  select ${columns.join(',')}
  from ${someTrigger ? 'table1': 'table2'}
  where 1=1 ${someThing ? ` and col = ${something}`}
`)
```

## another library

that being said there is [a library](https://github.com/felixfbecker/node-sql-template-strings) that allows you to do

```js
pg.query(SQL`SELECT author FROM books WHERE name = ${book} AND author = ${author}`)
```

but that's probably not going to be good enough to handle the complex type of join queries that tend to create these issues
