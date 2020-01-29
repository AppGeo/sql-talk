# Differences with other databases

## Field Lengths for strings

Field lengths don't actually save you any space, so just use the `text` type in postgres since `varchar(200)` or whatever is just a constraint that makes sure you don't but any strings that are too long into the field.

## primary keys

Primary keys are not really a thing, it's simple sugar for:
- Making an index on field
- which is a unique index
- and is restricted to not null

## Stored procedures

are not a thing in postgres despite what Andy says, they are called functions in postgres
