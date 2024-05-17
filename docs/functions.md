# functions

Honestly don't use them. Like seriously you should probably avoid them.  
Certain complex triggers that effect multiple rows or triggers are the exception but again they should probably be avoided because we don't use them very often which means

1. very hard to debug. We have very little insight into how they run and finding errors is hard
2. poor version control.  We don't have a great way right now to use functions, keep multiple databases up to date, and have manage changes to the functions
3. lack of experience, none of us don't know plpgsql very well. 

Functions can be written in just sql or they can be done in a procedural sql language called plpgsql.   Since you can only use plpgsql for triggers you should never be writing functions in sql[^1].

## Anatomy of a function

```sql
    CREATE OR REPLACE FUNCTION name(param1 param1Type, param2 param2Type)
    returns returnType as 'BEGIN return param1 = param2; end;' language 'plpgsql';
```

usually you'd want to use the fancy `$$` based text quoting so that you can use line breaks and quote marks in the function. here is a very basic one

```sql
 CREATE OR REPLACE FUNCTION challenge_type_compare(A text, B text)
    returns boolean as $$
    BEGIN
    return A = B;
    end;
    $$ language 'plpgsql';
```

you can also declare variables

```sql
 CREATE OR REPLACE FUNCTION challenge_type_compare(A text, B text)
    returns boolean as $$
    DECLARE
    C boolean;
    BEGIN
      C := A = B;
      return C;
    end;
    $$ language 'plpgsql';
```

note `:=` is used for assignment not `=`.

There are two ways to run queries, one for if it returns a row and one for if it doesn't.

Method 1


```sql
 CREATE OR REPLACE FUNCTION exampleThing(A text)
    returns text as $$
    DECLARE
    C text;
    BEGIN
       select field into C from table where column = A;
    
    if C is null then
      return 'boo';
    else
      return C;
    end if;
    end;
    $$ language 'plpgsql';
```

you can also do something like this with the execute command

```sql
execute $cwm$update challenges 
    SET area_id = $5,
    resolution_date = now()
    where 
    challenges.area_challenge_type = 'none' and
    challenges.location_id = $1 and
    challenge_type_compare(challenges.challenge_type, $2) AND
    (challenges.technology = $3 or (challenges.technology is null and $3 is null)) AND
    challenges.provider_id = $4 AND
    evidence_accepted_date is not NULL 
    and (disposition not in ('R', 'I') or disposition is null);
    $cwm$ using NEW.location_id, NEW.challenge_type, NEW.technology, NEW.provider_id, area_id_var;
```

`$cwm$` is a string delimiter I chose to avoid clashing with the outer $$ in the function.

## other notes

- Tables are types so if you pass `tablename.*` to a function, the tables parameter type can just be `tablename` its the same for variables you declare and returns
- you can loop through arrays, remeber to declare the loop variable then its `foreach x in array array_name loop ... end loop;`

[^1]: only a sith deal in absolutes, if you have good reason to do a non trigger function that takes these problems into account, don't let this stop you.