---
layout: post
title: "A Gentle Introduction to SQL and NoSQL Databases"
date: 2017-02-09
---

At some point during your career as a web developer, you will have to decide
what type of database is best suited for your application. You might face this
decision very soon, if you are planning on building your own application as a
side project. And at the very least, you will probably be asked about databases
in a job interview for a web developer role.

Web applications use either relational (SQL) or non-relational (NoSQL) databases
for persistent data storage. The point of this post is to provide a simple,
high-level overviews of relational and non-relational databases, highlight their
differences and discuss scenarios in which to use each, and briefly examine
common database management systems for each. I do not claim to be a database
expert, so some of the language in this post will use generalizations. I am
always looking to learn more about databases, but I know enough to be an
effective developer, and hopefully this post will provide you with enough info
so you can make more informed decisions regarding databases.

## Relational (SQL) Databases

In relational databases, the data is stored in tables. These tables are like
Excel spreadsheets with rows and columns, except a table in a database is
generally much more strict in its structure. A row in a table represents a
single record, and the columns are the attributes for that record. For example,
we might have a `blog_posts` table with various attributes for `title`, `body`,
and `date_published`. The table will also have an `id` column, which is a unique
number for each record in the table.

```
# blog_posts

| id | title       | body            | data_published |
|  1 | First Post! | blah blah...    | Feb 6, 2017    |
|  2 | Iguanas     | Yay iguanas...  | Feb 7, 2017    |
```

As you can imagine, data often has relationships with other data. That's where
the term "relational" comes from. In our `blog_posts` example, we might want to
also want to store the `author` for each blog post. We could store attributes
such as `author_first_name`, `author_last_name`, `author_email`, etc. with each
`blog_post` record, but that would get very messy very fast. Plus, there's
probably already a `users` table with all that information. So instead, we can
add an `author_id` column to our `blog_posts` table.

```
# blog_posts

| id | title       | body           | data_published | author_id |
|  1 | First Post! | blah blah...   | Feb 6, 2017    |        23 |
|  2 | Iguanas     | Yay iguanas... | Feb 7, 2017    |         5 |
```

And the `author_id` values match up to `id` values in the authors table.

```
# author

| id | first_name  | last_name | email                 |
|  5 | David       | Mitchell  | dmitch@cloudatlas.com |
| 23 | Margaret    | Atwood    | handmaid@tales.com    |
```

So when you retrieve a blog post, you can "join" the data with the corresponding
author data by matching the `author_id` and `id` columns from the tables.
Conversely, you can lookup an author and then "join" the blog post data to find
all the blog posts by a particular author.

SQL stands for "structured query language," the language used to interact with
relational databases.

## Non-Relational (NoSQL) Databases

Non-relational databases make up all the other types of databases that aren't
relational databases, and NoSQL means "not only SQL." This post will cover two
types of NoSQL databases, document stores and key-value stores.

### Document Stores

In document stores, the data is stored in collections, and collections are made
up of objects referred to as documents. These collections are the equivalent of
relational tables, and documents are the equivalent of records. So a `users`
collection would consist of documents, each one representing a different user.

Documents have attributes, which are defined for each collection. In the case of
the `users` collection, we might have attributes like `first_name`, `last_name`,
and `email`. Document objects are encoded using familiar formats such as JSON,
XML, or YAML. So JSON-encoded documents might look like:

```
# users

{
    "first_name": "David",
    "last_name": "Mitchell",
    "email": "dmitch@cloudatlas.com"
},
{
    "first_name": "Margaret",
    "last_name": "Atwood",
    "email": "handmaid@tales.com"
}
```

Document stores can also allow for relationships between documents through a
similar method as is done in relational databases, referencing another object's
`id`.

In the following example, there is some redundancy, but it illustrates two ways
a relationship could be defined. In the `authors` collection, documents contain
a `blog_posts` attribute which is an array of references to blog post documents.
In the `blog_posts` collection, the documents contain an `author_id` attribute
which is a reference to the `authors` collection. Based on the data you are
working with, a one-to-many or many-to-many relationship could be handled with
either method.

```
# authors

{
    "id": 2,
    "first_name": "David",
    "last_name": "Mitchell",
    "email": "dmitch@cloudatlas.com",
    "blog_posts": [1, 4]
}

# blog_posts

{
    "id": 1,
    "title": "First Post!",
    "body": "blah blah...",
    "date_published": "Feb 6, 2017",
    "author_id": 2
},
{
    "id": 4,
    "title": "Iguanas",
    "body": "Yay iguanas...",
    "date_published": "Feb 7, 2017",
    "author_id": 2
}
```

For some data, it may even make sense to define relationships between data by
embedding documents within documents. An example of this might be a shopping
cart:

```
# shopping_carts

{
    "user_id": 24,
    "items": [{
        "name": "Black Swan Green",
        "price": 15.99,
        "quantity": 2
    }, {
        "name": "Oryx and Crake"
        "price": 24.99,
        "quantity": 1
    }]
}
```

### Key-Value Stores

Key-value stores are simpler than document stores and are exactly what they
sound like, key-value pairs. Like big hashes. Some key-value stores are just
string keys paired with string values. Others allow for more complex data
structures for the values, such as unordered sets of strings.

A common use case for key-value stores in web applications is for storing user
preferences. User preferences often need to persist across user sessions, but
they may not be appropriate to be stored directly on the user record/document in
the database. Because key-value stores are generally made to be accessed
quickly, they provide a simple and fast storage solution for user preferences.

## ACID Compliance

An important topic to discuss when comparing relational and non-relational
databases is ACID. Acid stands for:

- **A**tomicity: Transactions are performed in an "all or nothing" manner. If
  one or more operations within the transaction fails, then all other operations
  in the transaction will fail. A common example is a banking transaction where
  money should be withdrawn from one bank account and deposited into another. If
  either of those operations fail, you want the entire transaction to fail.
- **C**onsistency: The data can only be modified in ways that are allowed and
  adhere to any and all defined constraints. An example of a constraint is that
  the `email` attribute is limited to 100 characters. In this case, consistency
  in the database means that a transaction will never result in an email with
  more than 100 characters.
- **I**solation: Concurrent transactions result in the data being in a state as
  if the transactions occurred one after the other. There's a bit more to it,
  but to keep it simple you can think of isolation as data being locked when it
  is the target of a transaction. When data is locked, other transactions are
  not operating on the data, and the data is isolated.
- **D**urability: Once a transaction is complete, the new state of the data will
  persist even if a system failure or some other error occurs immediately
  afterward.

Again, I'm speaking in generalizations, but it is commonly understood in web
development that relational databases are ACID-compliant while non-relational
databases are not fully ACID-compliant.

With full ACID compliance, there are some drawbacks. Most notably, performance
and scalability. Many NoSQL databases are designed to be more highly scalable
and faster than SQL databases, having sacrificed ACID-compliance.

## Deciding Between SQL and NoSQL Using the CAP Theorem

To help you evaluate the needs of your application in order to determine if you
should use a relational or non-relational database, you can reference the CAP
Theorem.

There's a [great video explanation](https://youtu.be/Jw1iFr4v58M) (only 4.5
minutes!) on YouTube that I recommend, but I'll try my hand at explaining it.

The CAP Theorem proposes that distributed database systems ("distributed"
meaning comprised of multiple nodes that communicate with each other to act as a
single system) have 3 potential attributes:

- **C**onsistency: When performing a "read" from one node in the system, you
  always receive the most recent "write", even if that "write" occurred on a
  different node. Different than the "consistency" in ACID.
- **A**vailability: When a request is made to a node, as long as the node has
  not failed, it will respond to the request.
- **P**artition Tolerance: When a node is removed from the system, the system
  continues to operate and uphold its other attributes.

However, the CAP Theorem also states that a distributed database system can have
a maximum of 2 of these attributes and it is theoretically impossible to have
all 3.

For example, if a "write" happens to one node in the system and is immediately
followed by a "read" to a different node, there are 3 possible outcomes of the
"read":

- **C & P**: The system is partition tolerant, so the nodes are not able to talk
  to each other. The node would wait until it is able to talk to the other nodes
  in order to maintain consistency before returning the data. However, since the
  nodes are not able to communicate, the node would not respond to the request
  and is thus not available.
- **A & P**: The system is partition tolerant, so the nodes are not able to talk
  to each other. The node would maintain availability and return the most recent
  version of the data that it has, even though it is not the most recent version
  of the data in the system. So the node is not consistent.
- **A & C**: The system is not partition tolerant and can only uphold its other
  attributes when all nodes are connected to the system. This means that the
  node would be able to communicate with the others nodes, maintain consistency
  by retrieving the most recent data, and maintain availability by responding to
  the request.

![CAP Theorem Diagram](/assets/cap_theorem.jpeg)

The third outcome can be dismissed because partition tolerance is an integral
aspect of distributed database systems. Therefore, the choice comes down to
consistency or availability.

From the section on ACID, we can generalize that since ACID-compliant database
systems are less performant than non-ACID-compliant database systems, then
ACID-compliant databases are less available than non-ACID-compliant systems.
They are less available but more consistent. Conversely, non-ACID-compliant
databases are more available and less consistent.

This choice between availability and consistency should be one of the factors
you consider when deciding between relational and non-relational databases. Is
it more important for your application to maintain very strict control over your
data, even at the cost of performance? Or is the speed of your application more
important than making sure your data is handled exactly as you expect 100% of
the time?

## Common Database Management Systems

This will be a short section about the database management systems that I see
most often in job listings, web applications I've worked on, web applications my
friends work on, etc.

### Relational

- [MySQL](https://www.mysql.com/): Very popular.
- [PostgreSQL](https://www.postgresql.org/): Implements more advanced data types
  than MySQL.
- [MariaDB](https://mariadb.org/): Community-developed fork of MySQL.
- [SQLite](https://www.sqlite.org/): Lightweight.

### Non-Relational

- [MongoDB](https://www.mongodb.com/): JSON-like document store.
- [CouchDB](http://couchdb.apache.org/): JSON document store.
- [Redis](https://redis.io/): Key-value store. Supports strings, lists, sets,
  hashes, and more.
- [Cassandra](http://cassandra.apache.org/): "Multiple master" model for high
  availability and scalability.
- [DynamoDB](https://aws.amazon.com/dynamodb/): Cloud data store from AWS.
- [Neo4j](https://neo4j.com/): Graph database.

---

For many web applications, the database provides the foundation upon which the
application is built. It is important to choose a database that will serve your
application and its users appropriately for the foreseeable future of your
application. However, as your application evolves, you can always re-evaluate
your database needs and migrate your data over to a new system.

And there you go! More than you ever probably wanted to read about databases in
the context of web development.
