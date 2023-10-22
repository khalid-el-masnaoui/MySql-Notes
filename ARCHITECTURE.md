# MySql Architecture

In-depth Notes about MySql , MySql/InnoDB architecture, ACID, connection and thread handling  and other MySql related concepts.

## MySQL Architecture

#### Introduction

To get the most from MySQL, you need to understand its design so that you can work with it, not against it. MySQL is flexible in many ways. For example, you can configure it to run well on a wide range of hardware, and it supports a variety of data types. However, MySQL's most unusual and important feature is its storage-engine architecture, whose design separates query processing and other server tasks from data storage and retrieval.

**MYSQL** is a relational database with a layered kind of architecture. The layers of the architecture include a server resource end at the middle, the storage engine at the bottom, and the client-end or query execution end at the top. It’s a three-layered architecture database system.

- The topmost layer contains the services that aren't unique to MySQL. They're services most network-based client/server tools or servers need: connection handling, authentication, security, and so forth.

- The second layer is where things get interesting. Much of MySQL's brains are here, including the code for query parsing, analysis, optimization, caching, and all the built-in functions (e.g., dates, times, math, and encryption). Any functionality provided across storage engines lives at this level: stored procedures, triggers, and views, for example.

- The third layer contains the storage engines. They are responsible for storing and retrieving all data stored "in" MySQL. The server communicates with them through the _storage engine API_. The API contains a couple of dozen low-level functions that perform operations such as "begin a transaction" or "fetch the row that has this primary key." The storage engines don't parse SQL or communicate with each other; they simply respond to requests from the server.


<p align="center">
<img src="./images/mysql_architecture.png"/>
</p>

#### Client

The Client sends request instructions to the Server. The client submits a request using valid MYSQL commands and expressions via Command Prompt or GUI screen. The output is shown on the screen if the expressions and commands are true. The following are some of the most relevant client layer services:

- **Connection Handling**- When a client sends a request to the server, the server accepts it and connects the client. When a client connects to the server at that time, the connection is given its own thread. All client-side queries are run with the aid of this thread.
- **Authentication**- When a client connects to an MYSQL account, authentication takes place on the server-side. The username, originating host, and password are used for authentication.
- **Security**- When a client connects successfully to MySQL server after authentication, the server verifies that the client has the permissions to run those queries against the MySQL server.

Each client connection gets its own thread within the server process. The connection's queries execute within that single thread, which in turn resides on one core or CPU. The server caches threads, so they don't need to be created and destroyed for each new connection.

When clients (applications) connect to the MySQL server, the server needs to authenticate them. Authentication is based on username, originating host, and password. Certificates can also be used across an Secure Sockets Layer (SSL) connection. Once a client has connected, the server verifies whether the client has privileges for each query it issues.

#### Server

The “Brain of MYSQL Architecture” is another name for this layer of the MYSQL system. When a client sends a request to the server, the server responds with output as soon as the instruction is matched. The following are the different sub-components of the MYSQL server:

- **Thread Handling**- When a client sends a request to the server, the server accepts it and connects the client. When a client connects to the server at that time, the connection is given its own thread. The thread management of the Server Layer provides this thread. The Thread Handling module also takes care of client-side queries that are run by the thread.
- **Parser**- A parser is a type of software component that creates a data structure (parse tree) from the given input. Lexical analysis is performed prior to parsing, in which the data is broken down into a number of tokens. After the data is accessible in smaller components, the parser performs Syntax and Semantics analysis and then generates a parse tree as an output.
- **Optimizer**- Once the parsing is complete, the Optimizer Block employs a variety of optimization techniques. These may include rewriting the query, determining the order in which it will read tables, choosing which indexes to use, and so on.
- **Query Cache**- Query Cache saves the entire result set for the query statement that was entered. MYSQL Server consults the query cache before parsing. When a client writes a query, if the query in the cache matches the query written by the client, the server skips parsing, optimization, and even execution and simply displays the output from the cache.
- **Buffer and Cache**- The cache and buffer will save the user’s previous query or issue. When a user types a query, it first goes to the Query Cache, which checks to see if the same query or problem exists in the cache. If the same question is open, it will produce results without interfering with the Parser and Optimizer.
- **Table Metadata Cache**- The metadata cache is a section of memory that stores information about databases, indexes, and objects. The metadata cache grows in size as the number of available databases, indexes, or objects grows.
- **Key Cache**- An index entry that uniquely identifies an object in a cache is known as a key cache.


#### Storage

The MYSQL database contains a different kind of storage engines which exist as a result of varying needs of databases. The storage engines are used to hold every user-created table in the database system. The **storage-end** facilitates the storing and retrieving of MYSQL data. The storage engine has an API that aids in the execution of the queries from the client end of the architecture just by passing rows back and forth in it.

**Note** :  MySQL’s default transactional storage engine and mostly used one is **InnoDB**, we will dicuss it in more details in the next section.

## InnoDB Storage Engine Architecture

InnoDB is MySQL’s default transactional storage engine, as well as the most important and widely used. It was created to handle a large number of short-lived transactions that are normally completed rather than rolled back. It’s also common for non-transactional storage because of its performance and automatic crash recovery. Unless you have a good reason to use a different engine, you can use InnoDB for your tables.

<p align="center">
<img src="./images/innodb_architecture.png"/>
</p>

#### InnoDB In-Memory Structures

###### Buffer Pool

The buffer pool is an area in main memory where `InnoDB` caches table and index data as it is accessed. The buffer pool permits frequently used data to be accessed directly from memory, which speeds up processing. On dedicated servers, up to 80% of physical memory is often assigned to the buffer pool.

For efficiency of high-volume read operations, the buffer pool is divided into pages that can potentially hold multiple rows. For efficiency of cache management, the buffer pool is implemented as a linked list of pages; data that is rarely used is aged out of the cache using a variation of the least recently used (LRU) algorithm.

By default, pages read by queries are immediately moved into the new sublist, meaning they stay in the buffer pool longer.

Knowing how to take advantage of the buffer pool to keep frequently accessed data in memory is an important aspect of MySQL tuning


###### Change Buffer

The change buffer is a special data structure that caches changes to secondary index pages when those pages are not in the buffer pool. The buffered changes, which may result from `INSERT`, `UPDATE`, or `DELETE` operations , are merged later when the pages are loaded into the buffer pool by other read operations.

In memory, the change buffer occupies part of the buffer pool. On disk, the change buffer is part of the system tablespace, where index changes are buffered when the database server is shut down.

###### Adaptive Hash Index

The adaptive hash index enables `InnoDB` to perform more like an in-memory database on systems with appropriate combinations of workload and sufficient memory for the buffer pool without sacrificing transactional features or reliability

Hash indexes are built on demand for the pages of the index that are accessed often. If `InnoDB` notices that queries could benefit from building a hash index, it does so automatically.

If a table fits almost entirely in main memory, a hash index speeds up queries by enabling _direct_ lookup of any element, turning the index value into a sort of pointer.

######  Log Buffer

The log buffer is the memory area that holds data to be written to the log files on disk. Log buffer size is defined by the `innodb_log_buffer_size`  variable. The default size is 16MB. The contents of the log buffer are periodically flushed to disk. A large log buffer enables large transactions to run without the need to write redo log data to disk before the transactions commit. Thus, if you have transactions that update, insert, or delete many rows, increasing the size of the log buffer saves disk I/O.

#### InnoDB On-Disk Structures

On disk-structures for InnoDB can be divided into the following : 

- Tables
- Indexes
- Tablespaces
- Doublewrite Buffer
- Redo Log
- Undo Logs

###### Tablespace

- ** _The System Tablespace_**  `fileName.ibdata1` :  is the storage area for the change buffer. It may also contain table and index data if tables are created in the system tablespace rather than file-per-table or general tablespaces. In previous MySQL versions, the system tablespace contained the `InnoDB` data dictionary. In MySQL 8.0, `InnoDB` stores metadata in the MySQL data dictionary. In previous MySQL releases, the system tablespace also contained the doublewrite buffer storage area. This storage area resides in separate doublewrite files as of MySQL 8.0.20.
- **_File-Per-Table Tablespaces_**  `fileName.ibd` : A file-per-table tablespace contains data and indexes for a single `InnoDB` table, and is stored on the file system in a single data file. `InnoDB` creates tables in file-per-table tablespaces by default. This behavior is controlled by the `innodb_file_per_table`  variable. Disabling `innodb_file_per_table` causes `InnoDB` to create tables in the system tablespace.