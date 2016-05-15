**Author: Jacob Mastel**

**Suggested Tags:** raspberrypi, data

# Storing Data on a Raspberry Pi

No matter what you do with your Raspberry Pi, you're bound to need to save information in some way or another. The main site has seen no shortage of questions on [databases](http://raspberrypi.stackexchange.com/search?q=databases) of various types. In this post, I'm going to discuss the various ways to store information on a Raspberry Pi, and the pros and cons of the various options.

## Flat File

### Definition

The simplest method of storing information is in what's called a *[flat file database](https://en.wikipedia.org/wiki/Flat_file_database)*. In short, a flat file database is a set of information that's written entirely to the file system as either a text or binary file. By definition, the entirety of the database is stored in a single file, though many people also refer to a set of files as a flat file database.

Of all the ways you could store information, flat files are usually one of the fastest out there. Since there is no server process involved, your biggest overhead usually comes from whatever library you may be using to save your information. If you're using a binary format, saving to a flat file usually just involves writing your object(s) directly, and if parsing the information into a text format, it's likely your language may have a native library built into it.

There are however, a few major downsides to using a flat file database. The primary being that they don't give you anything in the realm of querying capability. Search or logic that you'd normally get from a database will have to be implemented in your own code. While that's not a downside for many applications, it's a bigger draw back in others.

The other major drawback of using a flat file database is that only one process can use it at a given time, and they are not thread-safe without some kind of custom built mechanism.

### Examples

1. The original Microsoft Word format `.doc` was a binary file where the application simply wrote the document struct using a system `write()` call.
2. A text file containing data parsed as `xml`, `json`, `csv`, or a number of other formats. 

## Embedded Databases

Embedded databases are technically flat file databases. I have, however, separated them into a separate category. Generally speaking, an embedded database is saved as a flat, binary file, and does not require a separate process to manage it.

What differentiates them from a standard flat file database is that the libraries used to interact with them provide a significant subset of the features found in full database management systems (DBMS). SQLite for example allows for standard SQL queries, indexing, view creating, and multiprocessing support, among other things.

As a flat file system, you still have a low overhead as it lacks the management process found with a full DBMS. Their added functionality does add some overhead, but it's insignificant in comparison. Like other flat file databases though, the cache support offered by embedded databases is slim to none. 

### Examples

1. [SQLite](http://sqlite.org/) an embedded, relational database
2. [Unqlite](https://unqlite.org/) an embedded, noSQL, document store database

## Database Management Systems (DBMS)

A [DBMS](https://en.wikipedia.org/wiki/Database) is an application that stores and retrieves data. It runs as a standalone application that provides an API for other applications to interact with. These applications are usually very complex and offer powerful features that can make dealing with large amounts of data breeze.

Traditionally, databases are [relational](https://en.wikipedia.org/wiki/Relational_database), and most of what you'll find ready to use for the Raspberry Pi are in this classification. Although, the Raspberry Pi's performance and popularity has continued to grow, so there may well be additional options soon.

One of the biggest advantages of using a DBMS is that unlike the lack of caching abilities in the former options, a DBMS is able to keep the most used information in RAM between instances of your application running. If you're just logging sensor information, this may not be a big deal, but if you're trying to run a website that has a high amount of traffic, the advanced caching capabilities of an RDMS can make a huge difference.

Unfortunately, all that performance can come at a significant cost. The RDMS will take a non-insignificant amount of disk space and system resources. You'll have to decide if that trade off is worth it for your specific project. Alternatively, you can store the RDMS on a different machine, and connect to it via a network. This removes the load from the given Raspberry Pi, but does require a decent network connection. 

### Examples
1. [MySQL](https://www.mysql.com/) a relational DBMS
2. [PostgreSQL](http://www.postgresql.org/) a relational DBMS
3. [MongoDB](https://www.mongodb.com/) a noSQL document database

## In Summary

For most projects, flat file or embedded databases are probably sufficient, but some projects may benefit from more powerful solutions. You'll need to evalute what your project is doing, and determine which works for you.
