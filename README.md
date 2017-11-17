# Node Question ORM Insert Walkthrough

## Objectives

1. Define a `insert()` instance function for `Question`
2. Write an `INSERT INTO` SQL statement for `Question` instances.
3. Use SQL replacement for sanitation in your SQL statement.
4. Maintain the scope of the instance within the entire `insert()` function.
5. Return a `Promise` for the `insert()` function.
6. Execute a SQL Statement using the `sqlite3` NPM package with `db.run()`
7. Set the database returned primary key as the `id` property of the newly saved instance.

## Instructions

In `models/Question.js` there is an ORM `Question` class that implements migration method for its SCHEMA `.CreateTable()` and a `constructor` that defines a `content` property for instances.

The goal is to build an instance function `insert()` that can execute the necessary SQL to insert a quesion's content based on an instance of the class `Question` into a `questions` table in our database. 

In order to function properly, the database execution must be wrapped in a `Promise` that resolves. 

Maintain access to the instance of the question by casting it into a variable `self` early in the `insert()` function. Because this function will have multiple callbacks, the scope of `this` will change and this is the most elegant solution.

After the row is inserted, he callback of `db.run()` provides access to the newly inserted row through the scope of `this`. Use `this.lastID` to set the `id` property of the instance that was earlier cast into a variable `self`.

Resolve the promise by returning the updated instance, `self`.

**SPOILER: Below you will find Walkthrough Instructions for solving the lab**

## Walkthrough Instructions

**These instructions are progressive and if you follow them you will solve the lab. Only the final code block is the solution, all other code blocks are building up to it.**

Our `Question` class begins below.

**File: [models/Question.js](models/Question.js)**
```js
'use strict';

const db = require("../config/db")

class Question{
  static CreateTable() {
  return new Promise(function(resolve){
    const sql = `CREATE TABLE questions (
      id INTEGER PRIMARY KEY,
      content TEXT
    )`
    
    db.run(sql, function(){
      resolve("questions table created")
    })      
  })
  }

  constructor(content){
    this.content = content
  }
}


module.exports = Question;
```

The first thing to do is stub out our `insert()` instance function.

**File: [models/Question.js](models/Question.js)**
```js
const db = require("../config/db")

class Question{
  static CreateTable() {
    return new Promise(function(resolve){
      const sql = `CREATE TABLE questions (
        id INTEGER PRIMARY KEY,
        content TEXT
      )`
      
      db.run(sql, function(){
        resolve("questions table created")
      })      
    })
  }

  constructor(content){
    this.content = content
  }

  insert(){

  }
}

module.exports = Question;
```

Running the tests in this state produces:

```
  Question
    as a class
      .CreateTable()
        ✓ is a static class function
        ✓ returns a promise
        ✓ creates a new table in the database named 'questions'
        ✓ adds 'id' and 'content' columns to the 'questions' table
    insert()
      ✓ is a function
      1) returns a promise
      2) inserts the row into the database
      3) sets the id of the instance based on the primary key
      4) returns the instance as the resolution of the promise


  6 passing (42ms)
  2 failing

  1) Question
       insert()
         returns a promise:
         AssertionError: expected undefined to be an instance of Promise
```

Focusing only on the first error, we know that ultimately `insert()` should be returning a promise. Let's implement that as naively as possible.

**File: [models/Question.js](models/Question.js)**
```js
const db = require("../config/db")

class Question{
  static CreateTable() {
    return new Promise(function(resolve){
      const sql = `CREATE TABLE questions (
        id INTEGER PRIMARY KEY,
        content TEXT
      )`
      
      db.run(sql, function(){
        resolve("questions table created")
      })      
    })
  }

  constructor(content){
    this.content = content
  }

  insert(){
    return new Promise(function(resolve){
      resolve("This Does Nothing!")
    })
  }
}

module.exports = Question;
```

Running the tests again in the state above shows:

```
  Question
    as a class
      .CreateTable()
        ✓ is a static class function
        ✓ returns a promise
        ✓ creates a new table in the database named 'questions'
        ✓ adds 'id' and 'content' columns to the 'questions' table

    insert()
      ✓ is a function
      ✓ returns a promise
      1) inserts the row into the database
      2) sets the id of the instance based on the primary key
      3) returns the instance as the resolution of the promise

  6 passing (81ms)
  3 failing

  1) Question
       insert()
         inserts the row into the database:
         Uncaught TypeError: Cannot read property 'content' of undefined

```

While we are now returning a promise, there is no logic to actually insert the row into the database, thus the test fails when trying to compare the `content` property of the returned row to the expected value.

Let's first just write out the SQL, with sanitiation replacements, for our `INSERT`.

```sql
INSERT INTO questions (content) VALUES (?) 
```

The `?` represents the future value for the questions content, but we don't want to directly inject that into the SQL statement string as we will let the DB driver sanitize it.

With that we can implement the first part of the DB execution call with the `db.run()` function and callback.

**File: [models/Question.js](models/Question.js)**
```js
const db = require("../config/db")

class Question{
  static CreateTable() {
    return new Promise(function(resolve){
      const sql = `CREATE TABLE questions (
        id INTEGER PRIMARY KEY,
        content TEXT
      )`
      
      db.run(sql, function(){
        resolve("questions table created")
      })      
    })
  }

  constructor(content){
    this.content = content
  }

  insert(){
    const sql = `INSERT INTO questions (content) VALUES (?)`
    return new Promise(function(resolve){
      db.run(sql, [this.content], function(err, result){
        resolve("Row inserted!")
      })
    })
  }
}

module.exports = Question;
```

The ambition here is to call `db.run()`, pass it our `sql` statement, and pass it `this.content` to replace the `?` in the SQL for the instances actual content, referenced through `this`. The callback to `db.run()` will resolve the promise and let's see if the test for `1) inserts the row into the database` passes, it should.

```
  Question
    as a class
      .CreateTable()
        ✓ is a static class function
        ✓ returns a promise
        ✓ creates a new table in the database named 'questions'
        ✓ adds 'id' and 'content' columns to the 'questions' table
    insert()
      ✓ is a function
      ✓ returns a promise
(node:12608) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): TypeError: Cannot read property 'content' of undefined
(node:12608) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
(node:12608) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 3): TypeError: Cannot read property 'content' of undefined
      1) inserts the row into the database
(node:12608) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 5): TypeError: Cannot read property 'content' of undefined
      2) sets the id of the instance based on the primary key
      3) returns the instance as the resolution of the promise


  6 passing (4s)
  3 failing

  1) Question
       insert()
         inserts the row into the database:
     Error: Timeout of 2000ms exceeded. For async tests and hooks, ensure "done()" is called; if returning a Promise, ensure it resolves.
```

**Ahhhh!!!!** Everything blew up and we're getting all sorts of weird errors. This is a tuff one, not easy to debug. In these situations it's important to slow down, not panic, pull out your detective hat and magnifying glass, and start looking for clues.

`(node:12608) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): TypeError: Cannot read property 'content' of undefined`

While intimidating, it actually provides a clue. `TypeError: Cannot read property 'content' of undefined`. The promise we made in our `insert()` was unable to resolve because something in that code was calling `.content` on some object we thought existed but in fact, was `undefined`. Any error in javascript will interrupt the program, thus explaining why the promise was unresolved. The line `resolve("Row inserted!")` never actually ran. The only part of our code that mentions `.content` is `[this.content]`, so the real question is, why is `this` `undefined`?

Every function can create a new scope. When we create the promise, we create a new scope in the callback of the promise `function(resolve){}`. Within that function, the meaning of `this` changes. We expected `this` to be the instance of the question but it wasn't.

This is the nature of lots of errors, unmet expectations of the assumptions we litter throughout our code. How can we maintain access to the instance without relying on `this`? Watch:

**File: [models/Question.js](models/Question.js)**
```js
const db = require("../config/db")

class Question{
  static CreateTable() {
    return new Promise(function(resolve){
      const sql = `CREATE TABLE questions (
        id INTEGER PRIMARY KEY,
        content TEXT
      )`
      
      db.run(sql, function(){
        resolve("questions table created")
      })      
    })
  }

  constructor(content){
    this.content = content
  }

  insert(){
    const self = this // THIS IS THE CRUX
    const sql = `INSERT INTO questions (content) VALUES (?)`
    return new Promise(function(resolve){
      db.run(sql, [self.content], function(err, result){
        resolve("Row inserted!")
      })
    })
  }
}

module.exports = Question;
```

By casting `this` into a function or lexical scope variable `self`, that new variable will be available in every function nested and defined within `insert()`. As the definition of `this` changes within the various functions and callbacks, `self` will remain a consistent reference to the original defintion of `this`, the instance of the question itself.

We then replace the reference to `this.content` with a reference to `self.content`, which we know for sure will be the instance of the question. Let's run this code.

```
  Question
    as a class
      .CreateTable()
        ✓ is a static class function
        ✓ returns a promise
        ✓ creates a new table in the database named 'questions'
        ✓ adds 'id' and 'content' columns to the 'questions' table
    insert()
      ✓ is a function
      ✓ returns a promise
      ✓ inserts the row into the database
      1) sets the id of the instance based on the primary key
      2) returns the instance as the resolution of the promise

  7 passing (38ms)
  1 failing

    1) Question
       insert()
         sets the id of the instance based on the primary key:
         Uncaught AssertionError: expected undefined not to be undefined
         at Statement.<anonymous> (test/models/QuestionTest.js:107:18)
```

We're now getting one error, that the id of the instance should have been set to the id of the corresponding row in the database. We know we can get access to the `id` through the scope of `this` in `db.run()`, so that in our `db.run()` callback `this.lastID` is the ID of the primary key of the last inserted row. Additionally, `self` has already been cast to be the recently saved `question` instance, so we just need to set its `id` property to `this.lastID`.

**File: [models/Question.js](models/Question.js)**
```js
const db = require("../config/db")

class Question{
  static CreateTable() {
    return new Promise(function(resolve){
      const sql = `CREATE TABLE questions (
        id INTEGER PRIMARY KEY,
        content TEXT
      )`
      
      db.run(sql, function(){
        resolve("questions table created")
      })      
    })
  }

  constructor(content){
    this.content = content
  }

  insert(){
    const self = this // THIS IS THE CRUX
    const sql = `INSERT INTO questions (content) VALUES (?)`
    return new Promise(function(resolve){
      db.run(sql, [self.content], function(err, result){
        self.id = this.lastID
        resolve("Row Inserted!")      
      })
    })
  }
}

module.exports = Question;
```

With the final failing run of the test suite we'd see something like:

```
  Question
    as a class
      .CreateTable()
        ✓ is a static class function
        ✓ returns a promise
        ✓ creates a new table in the database named 'questions'
        ✓ adds 'id' and 'content' columns to the 'questions' table
    insert()
      ✓ is a function
      ✓ returns a promise
      ✓ inserts the row into the database
      ✓ sets the id of the instance based on the primary key
      1) returns the instance as the resolution of the promise


  8 passing (43ms)
  1 failing

  1) Question
       insert()
         returns the instance as the resolution of the promise:
         AssertionError: expected 'Row Inserted!' to deeply equal { Object (content, id) }
         at Context.<anonymous> (test/models/QuestionTest.js:116:35)
         at <anonymous>
```

We just need to make the promise `resolve` by returning the `question` instance, accessible through `self`.

**Solution**
**File: [models/Question.js](models/Question.js)**
```js
const db = require("../config/db")

class Question{
  static CreateTable() {
    return new Promise(function(resolve){
      const sql = `CREATE TABLE questions (
        id INTEGER PRIMARY KEY,
        content TEXT
      )`
      
      db.run(sql, function(){
        resolve("questions table created")
      })      
    })
  }

  constructor(content){
    this.content = content
  }

  insert(){
    const self = this // THIS IS THE CRUX
    const sql = `INSERT INTO questions (content) VALUES (?)`
    return new Promise(function(resolve){
      db.run(sql, [self.content], function(err, result){
        self.id = this.lastID
        resolve(self)      
      })
    })
  }
}

module.exports = Question;
```

All green!

```
  Question
    as a class
      .CreateTable()
        ✓ is a static class function
        ✓ returns a promise
        ✓ creates a new table in the database named 'questions'
        ✓ adds 'id' and 'content' columns to the 'questions' table
    insert()
      ✓ is a function
      ✓ returns a promise
      ✓ inserts the row into the database
      ✓ sets the id of the instance based on the primary key
      ✓ returns the instance as the resolution of the promise


  9 passing (42ms)
```

Yay!