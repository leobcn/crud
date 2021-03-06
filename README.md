## CRUD

A minimalistic relational database library for Go, with simple and familiar interface. [Why?](#why-another-ormish-library-for-go)

* [Install](#install)
* [Initialize](#initialize)
* [Define](#define)
  * [Create & Drop Tables](#create--drop-tables)
  * [Reset Tables](#reset-tables)
  * [SQL Options](#sql-options)
* CRUD:
  * [Create](#create)
  * [Read](#read)
    * [Reading a single row](#reading-a-single-row)
    * [Reading multiple rows](#reading-multiple-rows)
    * [Scanning to custom values](#scanning-to-custom-values)
  * [Update](#update)
  * [Delete](#delete)
  * [Transactions](#transactions)
* [Logs](#logs)
* [Custom Queries](#custom-queries)
* [Running Tests](#running-tests)
* [Why another ORMish library for Go?](#why-another-ormish-library-for-go)
* [Apps Using CRUD](#apps-using-crud)
* [What's Missing?](#whats-missing)

## Install

```bash
$ go get github.com/azer/crud
```

## Initialize

```go
import (
  "github.com/azer/crud"
  _ "github.com/go-sql-driver/mysql"
)

var DB *crud.DB

func init () {
  var err error
  DB, err = crud.Connect("mysql", os.Getenv("DATABASE_URL"))
  err = DB.Ping()
}
```

## Define

```go
type User struct {
  Id int `sql:"auto-increment primary-key"`
  FirstName string
  LastName string
  ProfileId int
}

type Profile struct {
  Id int `sql:"auto-increment primary-key"`
  Bio string `sql:"text"`
}
```

CRUD will automatically convert column names from "FirstName" (CamelCase) to "first_name" (snake_case) for you. You can still choose custom names though;

```go
type Post struct {
  Slug string `sql:"name=slug_id varchar(255) primary-key required"`
}
```

If no primary key is specified, CRUD will look for a field named "Id" with int type, and set it as auto-incrementing primary-key field.

##### Create & Drop Tables

`CreateTables` takes list of structs and makes sure they exist in the database.

```go
err := DB.CreateTables(User{}, Profile{})

err := DB.DropTables(User{}, Profile{})
```

##### Reset Tables

Shortcut for dropping and creating tables.

```go
err := DB.ResetTables(User{}, Profile{})
```

##### SQL Options

CRUD tries to be smart about figuring out the best SQL options for your structs, and lets you choose manually, too. For example;

```go
type Tweet struct {
 Text string `sql:"varchar(140) required name=tweet"`
}
```

Above example sets the type of the `Text` column as `varchar(140)`, makes it required (`NOT NULL`) and changes the column name as `tweet`. 

Here is the list of the options that you can pass;

* Types: `int`, `bigint`, `varchar`, `text`, `date`, `time`, `timestamp`
* `auto-increment` / `autoincrement` / `auto_increment`
* `primary-key` / `primarykey` / `primary_key`
* `required`
* `default='?'`
* `name=?`

If you'd like a struct field to be ignored by CRUD, choose `-` as options:

```go
type Foo struct {
 IgnoreMe string `sql:"-"`
}
```

## Create

```go
user := &User{1, "Foo", "Bar", 1}
err := DB.Create(user)
```

## Read

You can read single/multiple rows, or custom values, with the `Read` method.

##### Reading a single row:

```go
user := &User{}
err := DB.Read(user, "WHERE id = ?", 1) // You can type the full query if preferred.
// => SELECT * FROM users WHERE id = 1

fmt.Println(user.Name)
// => Foo
```

##### Reading multiple rows:

```go
users := []*User{}

err := DB.Read(&users)
// => SELECT * FROM users

fmt.Println(len(users))
// => 10
```

##### Scanning to custom values:

```go
names := []string{}
err := DB.Read(&names, "SELECT name FROM users")
```

```
name := ""
err := DB.Read(&name, "SELECT name FROM users WHERE id=1")
```

```go
totalUsers := 0
err := DB.Read(&totalUsers, "SELECT COUNT(id) FROM users"
```

## Update

Updates matching row in database, returns `sql.ErrNoRows` nothing matched.

```go
user := &User{}
err := DB.Read(user, "WHERE id = ?", 1)

user.Name = "Yolo"
err := DB.Update(user)
```

## Delete

Deletes matching row in database, returns `sql.ErrNoRows` nothing matched.

```go
err := DB.Delete(&User{
  Id: 1
})
```

## Transactions

Use `Begin` method of a `crud.DB` instance to create a new transaction. Each transaction will provide you following methods;

* Commit
* Rollback
* Exec
* Query
* Create
* Read
* Update
* Delete

```go
tx, err := DB.Begin()

err := tx.Create(&User{
  Name: "yolo"
})

err := tx.Delete(&User{
  Id: 123
})

err := tx.Commit()
```

## Logs

If you want to see crud's internal logs, specify `crud` in the `LOG` environment variable when you run your app. For example;

```
$ LOG=crud go run myapp.go
```

[(More info about how crud's logging work)](http://github.com/azer/logger)

## Custom Queries

````go
result, err := DB.Query("DROP DATABASE yolo") // or .Exec
````

## Running Tests

```bash
DATABASE_URL="?" go test ./...
```

## Why another ORMish library for Go?

* Simplicity, taking more advantage of `reflect` library to keep the API simple.
* Building less things with more complete abstractions
* Handling errors in an idiomatic way
* Good test coverage
* Modular & reusable code
* Making less unsafe assumptions. e.g: not mapping structs to SQL rows by column index.

## Apps Using CRUD

[Listen Paradise](http://listenparadise.org) is built with CRUD and [it's open source](http://github.com/azer/radio-paradise).

## What's Missing?

* **Migration:** We need a sophisticated solution for adding / removing columns when user changes the structs.
* **Explicit Read Methods:** We can have explicit alternatives of `Read` method for people who prefers.
* **Testing Transactions:** Transactions work as expected but there is a sync bug in the test causing failure. It needs to be fixed.
* **Comments:** I like self-documenting code and nice README's rather than commenting code, so rarely comment my code.
* **Hooks:** I'm not sure if this is needed, but worths to consider.
