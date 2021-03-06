---
layout: post
title: "The Basics of Database Indexes for Relational Databases"
date: 2017-08-23
---

The purpose of creating an index on a particular table in your database is to
make it faster to search through the table and find the row or rows that you
want. The downside is that indexes make it slower to add rows or make updates to
existing rows for that table. So adding indexes can increase read performance
and decrease write performance. Indexes are also used to enforce uniqueness
constraints, but I won't go into that for this post.

## But First, Let's Talk About Karaoke!

Besides being my favorite pastime (just FYI, I'm Filipino), karaoke provides a
good analogy for indexes and might help you when reading the rest of this post.

All karaoke joints I've been to provide songbooks that contain the list of songs
you can sing. The pages are organized like a database table, usually with 3
columns: song title, artist, and code. The code is what you enter into the
system to bring up the karaoke track. And there are usually two songbooks: one
sorted by artist name and one sorted by song title. That way you can either
think of an artist you want to sing and then look through their songs, or look
up a specific song title you know you want to sing or in case you don't know the
artist's name. These songbooks are like indexes for the database of songs. They
provide a sorted list of data that is easily searchable by relevant information.
There isn't an index for the code column because that information is not
relevant information that you would search by.

Back to actual databases…

## How Do Indexes Increase Read Performance?

Think about the primary key column of a particular table. It is usually the `id`
column, and it is usually a digit that increments with each new row in the
table. So when you try to retrieve a specific row from the database using its
`id`, the database doesn't need to search through every single row to find the
one you're asking for; the data is already sorted and can be searched using
efficient algorithms. Even if you were to rearrange the rows in the table so
that the `id` column was unsorted, the primary key column of tables is
automatically indexed, meaning there is a copy of that column with the sorted
data that the database will use to search. And that index contains pointers to
the actual rows in the table, so once it has found the correct `id` from that
copied data, it knows exactly where to find the rest of the information for that
row.

Indexes on columns that are not the primary key column work in the same way. For
example, you might have an articles table in your database that contains columns
for `title`, `body`, and `published_at`, as well as the primary key `id`.
Depending on how you query for data in your application, you might want to add
indexes on some of these columns to improve read performance.

If your application has a feature where you can search for articles by their
title, it might be wise to put an index on the `title` column. This will create
a copy of that column where all the articles' titles are sorted. Maybe your
application also allows users to view articles from a specific time period that
they define. Instead of the database having to check every single row's
`published_at` column, you can put an index on that column, and the database
will be able to easily find the articles that were published in that time frame
because they will all be right next to each other. However, it seems unnecessary
(and a bit ridiculous) to put on index on the `body` column because your
application is unlikely to query the `articles` table by the full contents of
each article. (There are much better ways to search through large bodies of text
in databases.)

Indexes can also be useful for foreign key columns when dealing with
associations. Let's say the articles table also contains an `author_id` column
that corresponds to the `id` column on the users table. If you put an index on
the `author_id` column, when you query the database for all the articles by a
particular author, the results can be found much faster because all articles by
that author will be grouped together.

## Indexes On Multiple Columns

You can also create a single index from multiple columns in a table. Extending
the example from above, maybe your application allows authors to view their own
articles, and the default view is to show them in reverse chronological order.
On your `articles` table, it would make sense to create an index using both the
`author_id` and `published_at` columns. Here's a diagram of how you might think
about that index:

```
| author_id | published_at |
|-----------|--------------|
|           | 2017-08-17   |
|     1     | 2017-08-20   |
|           | 2017-08-22   |
|-----------|--------------|
|           | 2017-08-14   |
|           | 2017-08-20   |
|     2     | 2017-08-21   |
|           | 2017-08-22   |
|           | 2017-08-23   |
|-----------|--------------|
|     3     | 2017-08-01   |
```

Now when querying the database for all the articles by an author and ordered by
their `published_at` dates, the database can use this index to quickly retrieve
the data.

When creating an index from multiple columns, it is important that you specify
the correct order of the columns. The above index was created by first
specifying the `author_id` column and then the `published_at` column. Had it
been reversed, the index would not be nearly as useful for the page we want:

```
| published_at | author_id |
|--------------|-----------|
| 2017-08-01   | 3         |
|--------------|-----------|
| 2017-08-07   | 3         |
|--------------|-----------|
| 2017-08-14   | 2         |
|              | 3         |
|--------------|-----------|
| 2017-08-17   | 1         |
|--------------|-----------|
| 2017-08-20   | 1         |
|              | 2         |
|--------------|-----------|
| 2017-08-21   | 2         |
|--------------|-----------|
| 2017-08-22   | 1         |
|              | 2         |
|--------------|-----------|
| 2017-08-23   | 2         |
```

When creating an index, you want the database to be able to eliminate as many
items as possible for at each step of its search. In the first example, after
finding the appropriate `author_id`, all other rows for other authors are
eliminated, and the database then just has to search through a handful of
remaining rows. The second index is not even usable for our purposes of showing
a single author's articles in reverse chronology because there is no way to
effectively search through the indexed data. So keep in mind the column order
that will create the index that will be most beneficial to your database
queries.

## How Do Indexes Decrease Write Performance?

The cost of improving database read times using indexes is that write times
suffer. Adding a new row to a table without indexes is simple. The database
finds the next available space in the table to add the new entry and adds it,
that's it. However, when adding a new row to a table with one or more indexes,
the database adds the new entry to the table, and then it has to add a new entry
into each index on that table, making sure to insert the entry into the correct
spot in the index to ensure that the data is properly sorted. And this
performance degradation applies to creates, updates, and deletes for the table.
For this reason, adding unnecessary indexes on tables should be avoided, and
indexes that are no longer used should be removed.

Adding indexes is about improving performance of search queries. Maybe the goal
of your database is to provide a data store that is often written to and rarely
read from. If that is the case, decreasing the performance of the more common
operation, writing, is probably not worth the increase in performance you get
from reading.
