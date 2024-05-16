---
layout: post
title: SQL injections near us
---

Today let's talk a little about more practical security staff. SQL injections is an old kind of security vulnerability
that allows malicious user to execute arbitrary SQL script on you backend. It often causes because of lack of validation
on url query or path parameters that can be applied to the SQL query on your backend. Nowadays, most of popular
frameworks implements a proper validation during SQL construction but if you will try to construct the script by
yourself (without using of popular DAO code generators or built-in SQL library methods) you can miss such validation and
create a big hole in your security.

Let's take a look on the following example:

```go
stmt := squirrel.Select("*").From("public.users")

params := unsecuresqllib.OffsetPageParams{
    Limit:      1,
    Order:      "desc",
    PageNumber: 0,
}

// unsecure ApplyTo method with arguments "sql statement" and "column to apply".
// The last argument causes vulnerability if not validated.
stmt = params.ApplyTo(stmt, "id asc; delete from public.users where id=1; select * from public.users order by id")

str, _, _ := stmt.ToSql()
fmt.Println(str)

// User represents a row from 'public.users'.
type User struct {
    ID   uint64 `db:"id" json:"id" structs:"-"`        // id
    Name string `db:"name" json:"name" structs:"name"` // name
}

var users []User
if err := db.SelectContext(context.TODO(), &users, stmt); err != nil {
    panic(err)
}
```

For the table `users` with only two columns using an unsecure library (I won't tell you which one) the simple _SELECT_
query can be modified by applying order params to drop the entry from this table. This code snippet applies some params
to some statement and some column but the column value is not validated anywhere in the code. Under the hood library
executed `stmt = stmt.OrderBy(fmt.Sprintf("%s %s", col, p.Order))`, so the final query that will be submitted to the
database will
be: `SELECT * FROM public.users ORDER BY id asc; delete from public.users where id=1; select * from public.users order by id desc LIMIT 1 OFFSET 0`
