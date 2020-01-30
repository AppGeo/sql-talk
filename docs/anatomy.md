# Anatomy of a Query

## select

```sql
SELECT
  -- column names, possibly with functions called on them
  name,
  length,
  length/3 as length_yards
FROM
  -- list of tables to query from, this is where they are defined
  table_name,
  -- can also have full queries in here
  (
    SELECT
      *
    FROM
    -- this can be repeated
      (
        SELECT
          *
        FROM
          some_table
      )
      as bar
    WHERE
      field = '3'
  )
  as foo
WHERE
  -- queries go here
  field = value
  and
  -- where clauses can also reference queries
  otherField in (select field from table where foo = 'bar')
  -- no need for an as there
-- orderby groupby and limit go here
```


## update

### basic

```sql
UPDATE
  tablename
SET
  length_yards = length_feet/3,
  length_furlongs = length_feet/660
WHERE
  length_yards is null
OR
  length_furlongs is null;
```

### join
```sql
UPDATE
  tablename
SET
  field = 'value',
  tablename.otherField = otherTable.value
  -- only if other table also has that field
FROM
  otherTable
  -- other needed if you have a join
where
  tablename.id = otherTable.id
  --join them
  and
  tablename.value = ?
  -- if you need to specify which fields to update
```

## Delete

```sql
DELETE FROM
  tablename
WHERE
  foo = ?;
-- where is optional, though remember truncate
```

## Insert

## basic

```sql
INSERT INTO
  TABLE
  (text, number)
VALUES
  ('foo', 1),
  ('bar', 2);
```

## based on selection

```sql
INSERT INTO
  TABLE
  (text, number)
SELECT
  name as text,
  id as number
FROM
  otherTable
RETURNING
  text;
```

## Create table as

### copy a table

```sql
CREATE TABLE
  new_table
AS
TABLE
  old_table;
```

### copy table structure

```sql
CREATE TABLE
  new_table
AS
TABLE
  old_table
WITH NO DATA;
```

### create a table froma  query

```sql
CREATE TABLE
  new_table
AS
SELECT
  field1 as thingy,
  field2
FROM
  old_table
WHERE
  num BETWEEN 1 AND 5;
```
