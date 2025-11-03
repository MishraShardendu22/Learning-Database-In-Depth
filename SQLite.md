# SQLite

SQLite is a popular, lightweight, and self-contained relational database management system (RDBMS) with several distinguishing features.

## Serverless Architecture

Unlike traditional RDBMS like MySQL or PostgreSQL, SQLite doesn't require a separate server process to operate. The entire database is stored in a single file on disk, and applications read and write directly from these database files without intermediary server processes. This eliminates the need to install, setup, configure, initialize, manage, and troubleshoot server processes.

## Zero Configuration

SQLite requires no administrative setup or server configuration. No installation procedure, no server process management, and no configuration files are needed. Any program with disk access can use an SQLite database without administrative support. The system requires no setup procedure, no recovery actions after crashes, and works immediately out of the box.

## Embedded Database

SQLite is embedded within the application that uses it as a library that gets directly integrated into the application. The database runs in the same process as the application, reducing latency by eliminating inter-process communication overhead. The linker integrates the SQLite library—statically or dynamically—into applications which access SQLite's functionality through simple function calls.

## Small Footprint

The library size of SQLite is small, and the database file format is compact, making it well-suited for embedded systems, mobile applications, and applications where resources are limited.

## Cross-Platform

SQLite is highly portable and works across various operating systems, including Windows, Linux, macOS, Android, and iOS. The database files are platform-independent, meaning a database created on one platform can be used on another without modification.

## Transactional Support

SQLite is fully ACID-compliant, supporting atomic transactions, even across multiple tables. This ensures data integrity and reliability in case of crashes or power failures. All changes and queries are atomic, consistent, isolated, and durable, with guarantees maintained even if transactions are interrupted by program crashes, system failures, or power outages.

## Self-Contained

SQLite is self-contained, meaning it has no external dependencies. All the code needed to use SQLite is included in a single library, making it easy to integrate and use.

## Read-Only and In-Memory Modes

SQLite supports read-only database files and in-memory databases, which are particularly useful for temporary data storage, testing, or scenarios where data persistence is not required. Multiple connections can access in-memory databases using shared cache mode.

## Rich Feature Set

Despite its small size, SQLite offers a rich set of features, including support for most SQL syntax, views, triggers, and indexes. It also provides full-text search (FTS) and JSON support, making it versatile for a wide range of applications.

## Scalability for Small to Medium Applications

While not designed for high-concurrency, large-scale enterprise applications, SQLite can handle moderate workloads, making it a good choice for many desktop, mobile, and embedded applications. Write operations are performed sequentially due to file-level locking, though write-ahead logging (WAL) mode enables concurrent reads and writes.

## Public Domain

SQLite is in the public domain, which means it can be used freely for any purpose, including commercial applications, without the need for licensing fees.

These specialties make SQLite a popular choice for developers looking for a simple, yet powerful database solution for embedded systems, mobile applications, and small to medium-sized projects.

Documentation: <https://sqlite.org/fasterthanfs.html>

---

## Client Server Architecture Comparison

Client-server databases (PostgreSQL, MySQL, MongoDB, etc.) use separate deployed codebases.

Server: Runs as a standalone process (daemon). Manages storage, queries, and connections.
Client: Uses libraries or drivers (e.g., libpq for Postgres) to send SQL commands over TCP/IP or Unix sockets.

PostgreSQL has a separate server binary (postgres) and client tools/libraries (psql, libpq). The server can be deployed independently, and clients connect remotely via network.

## SQLite Vs MySQL

SQLite and MySQL are both popular relational database management systems (RDBMS), but they serve different purposes and have distinct characteristics

### Architecture

## **SQLite**: SQLite is a serverless database, meaning it does not require a separate server process to operate. The database is stored in a single file, and the database engine is embedded directly into the application. SQLite is self-contained and requires no external dependencies, making it easy to deploy and use within an application

## **MySQL**: MySQL follows the client-server model, where the database is managed by a server process that handles multiple client connections. The database and application run as separate processes. MySQL supports networked access, allowing multiple clients to connect to the database over a network

### Use Cases

## **SQLite**: Best suited for embedded systems, mobile applications, small to medium-scale applications, testing, and development environments. Ideal for single-user applications or scenarios where simplicity, low resource usage, and minimal configuration are important

## **MySQL**: Suitable for large-scale, multi-user, and high-concurrency applications, including web applications, enterprise systems, and large data-driven applications. Used in scenarios where a more robust and scalable solution is required, with support for high availability, replication, and clustering

### Performance

## **SQLite**: SQLite is very fast for read and write operations, especially in situations where the database size is small and concurrency is low. It excels in scenarios where the database is accessed by a single user or a few users. Limited concurrency support, as SQLite uses database-level locking, which can cause contention and slow performance when multiple users or processes try to write to the database simultaneously

## **MySQL**: MySQL is also fast, particularly for read-heavy workloads. It supports complex queries, indexing, and optimization techniques that can handle large datasets efficiently. Better concurrency support with features like table-level and row-level locking, making it more suitable for high-concurrency environments with multiple users accessing and modifying the database simultaneously

### Features

## **SQLite**: The entire database is stored in a single file, which is portable and easy to manage. Fully ACID-compliant with transactional support, ensuring data integrity. SQLite supports most SQL features but lacks some advanced features found in MySQL, such as stored procedures, triggers, and advanced indexing

## **MySQL**: Supports advanced features like stored procedures, triggers, views, replication, clustering, and partitioning, making it a more powerful RDBMS. MySQL offers built-in full-text search capabilities, especially with the InnoDB and MyISAM storage engines. MySQL supports different storage engines (e.g., InnoDB, MyISAM) that allow for different data handling and performance characteristics

### Scalability

## **SQLite**: Limited scalability, as it is designed for small to medium-sized databases. Not suitable for very large datasets or high-concurrency applications. Single-file database means the entire database is limited to a single file, which can become a bottleneck as the size of the database grows

## **MySQL**: High scalability, capable of handling very large databases with millions of rows and high levels of concurrency. Supports horizontal scaling through sharding and vertical scaling by adding more resources. Distributed database support through MySQL configurations in master-slave, master-master replication setups, and clustered environments to support large-scale applications

### Administration and Management

## **SQLite**: Requires minimal setup and administration. Ideal for developers who want to avoid the complexities of managing a separate database server. SQLite does not have built-in user management, roles, or access control features

## **MySQL**: Requires more setup and maintenance but offers robust tools for database administration, including user management, backups, security, and performance tuning. Supports multiple users with different permissions and roles, providing fine-grained access control

### Portability

## **SQLite**: Highly portable, as the database is stored in a single file, making it easy to move across different systems and platforms. The same database file can be used across different operating systems without modification

## **MySQL**: While MySQL is cross-platform, the database itself requires a server to be set up on the target platform, and data migration between platforms might require additional steps

### Licensing

## **SQLite**: SQLite is in the public domain, meaning it can be used freely for any purpose, including commercial applications, without licensing fees

## **MySQL**: MySQL is available under the GNU General Public License (GPL) for open-source projects. Commercial licenses are available for proprietary applications

### Summary

## **SQLite** is best for lightweight, low-concurrency, and embedded applications where simplicity, ease of use, and minimal configuration are key

## **MySQL** is better suited for large, high-concurrency, multi-user applications where scalability, advanced features, and robust management tools are required

## SQLite Internal Architecture

SQLite's architecture consists of several key components that work together to provide a complete database solution

### Storage Format

SQLite stores the entire database (schema, data, and indexes) in a single cross-platform file on disk. This file is in a binary format and can be easily copied, backed up, or transferred between systems

### SQLite Library

SQLite operates as a library that is embedded directly into the application that uses it. This library provides all the functionality needed to create, access, and manage the database without requiring an external database server

### Components of SQLite Architecture

## **A. SQL Compiler**: The SQL compiler is responsible for parsing SQL statements provided by the user or application. It converts the SQL text into a parse tree, which represents the logical structure of the SQL command

## **B. Virtual Database Engine (VDBE)**: After the SQL compiler generates the parse tree, the VDBE takes over. The VDBE is a virtual machine within SQLite that interprets the parse tree and executes the SQL commands. It generates bytecode from the parse tree, which it then executes to perform the database operations (such as querying, inserting, updating, or deleting data)

## **C. Backend (B-Tree)**: SQLite uses a B-tree data structure to store and manage database tables and indexes. The B-tree is highly optimized for read and write operations, and it supports efficient data retrieval and storage. Each table and index is stored as a B-tree, where the keys are the row IDs or index keys, and the values are the actual data or references to it

## **D. Pager**: The pager module is responsible for managing the database file on disk. It handles reading and writing pages (fixed-size blocks of data) from the database file and ensures that these operations are atomic and durable. The pager also manages the database's cache, which stores recently accessed memory pages to speed up read and write operations

## **E. Transaction Manager**: SQLite is fully ACID-compliant, and the transaction manager plays a key role in ensuring this. It manages transactions by maintaining a journal file, which logs changes to the database. This journal is used to roll back incomplete transactions in case of a crash or power failure, ensuring the integrity of the database

## **F. Shared Cache**: In cases where multiple database connections are used, SQLite can employ a shared cache mechanism. This allows multiple connections to share the same cache, improving performance and reducing memory usage. However, SQLite typically operates in single-threaded mode, with each connection having its private cache

## **G. OS Interface (VFS - Virtual File System)**: The Virtual File System (VFS) layer abstracts the underlying operating system's file system. It provides an interface for SQLite to perform file I/O operations, such as reading and writing to the database file, locking, and managing the file's lifetime. This abstraction allows SQLite to be highly portable across different operating systems

## **H. Locking and Concurrency Control**: SQLite uses a simple locking mechanism to control access to the database file and ensure consistency. It supports multiple readers but only one writer at a time. The locking modes in SQLite include

- **UNLOCKED**: No locks are held
- **SHARED**: Allows multiple readers to access the database simultaneously
- **RESERVED**: Prepares for a write operation while allowing ongoing reads
- **PENDING**: Indicates that a write operation is imminent, blocking new readers
- **EXCLUSIVE**: Full access to the database for writing, blocking all other operations

### SQLite File Format

SQLite databases are stored in a single file, which contains

## **Header**: Contains metadata about the database, including its page size, schema version, and more

## **Page Structure**: The database file is divided into pages, typically 4096 bytes in size. Each page can be used for various purposes, such as storing table data, index data, or free space

## **B-Tree Pages**: These pages store the B-tree structures for tables and indexes

## **Freelist Pages**: Pages that are not currently used are placed on a free list and can be reused for future allocations

### SQL Execution

When an application sends a SQL query to SQLite, the following steps occur

1. **Parsing**: The SQL text is parsed into a parse tree
2. **Bytecode Generation**: The VDBE generates bytecode based on the parse tree
3. **Execution**: The bytecode is executed by the VDBE to perform the desired database operations
4. **Result Handling**: Results are returned to the application or errors are reported if the execution fails

### ACID Properties

**Atomicity**: Ensures that transactions are fully completed or not done at all
**Consistency**: The database remains in a consistent state before and after a transaction
**Isolation**: Transactions are isolated from each other, preventing interference
**Durability**: Once a transaction is committed, it remains so, even in the case of a system crash

.](image-2.png)

## SQLite Parser

The SQLite parser is a crucial component of the SQLite database engine responsible for interpreting and analyzing SQL commands provided by the user or application. The parsing process is the first step in executing an SQL command, where the SQL text is transformed into a data structure that can be further processed by the database engine

### Key Concepts of the SQLite Parser

## **1. Parsing SQL Statements**: When an SQL statement is passed to SQLite, the parser's job is to read the text of the SQL statement and break it down into its constituent parts, such as keywords (SELECT, INSERT, UPDATE, DELETE, etc.), table names, column names, expressions, and values. The parser checks the SQL statement for syntactical correctness, and if the SQL command doesn't follow the correct syntax, an error is raised

## **2. Tokenization**: The first step in parsing involves tokenization, where the SQL statement is split into tokens. Each token represents a basic element of the SQL command, such as a keyword, identifier (like table or column names), or literal value. Tokenization is done by a lexer, which is part of the SQLite parser. The lexer scans the SQL statement and generates a stream of tokens that represent the different components of the SQL statement

## **3. Grammar and Syntax Rules**: The SQLite parser uses a predefined set of grammar rules to understand and validate SQL commands. These rules are defined in a language called Bison (a parser generator) and are based on the SQL standard. The parser applies these rules to the token stream to ensure that the SQL statement adheres to the correct syntax

## **4. Parse Tree**: After successfully parsing the SQL statement, the parser generates a parse tree (also known as a syntax tree). The parse tree is a hierarchical representation of the SQL statement, where each node represents a different part of the SQL command. For example, in a SELECT statement, the root of the tree might represent the SELECT operation, with child nodes representing the columns to be selected, the tables involved, and any conditions or joins

## **5. Abstract Syntax Tree (AST)**: In some cases, the parse tree is further transformed into an Abstract Syntax Tree (AST), which is a more abstract representation of the SQL command. The AST is simpler and more streamlined, removing unnecessary details while retaining the essential structure needed for execution

## **6. Virtual Database Engine (VDBE) Bytecode Generation**: Once the parse tree or AST is generated, it is handed over to the Virtual Database Engine (VDBE), which converts it into a series of bytecode instructions. These instructions are then executed by the VDBE to perform the desired database operations. The VDBE bytecode is what interacts with the database tables, indexes, and data to fulfill the SQL command

## **7. Error Handling**: The parser is also responsible for error handling. If an SQL statement is not valid, the parser generates an error message indicating the nature of the problem, such as a syntax error, missing keyword, or invalid table name

### Yacc Grammar File

The SQLite Yacc grammar file is a crucial part of SQLite's source code. It defines the syntax of the SQL language that SQLite supports, specifying how SQL statements should be parsed into their constituent components (such as keywords, expressions, and clauses). This file is typically written in a language like Yacc or Bison, which are tools used to generate parsers

### Overview of Yacc/Bison

## **Yacc (Yet Another Compiler-Compiler)**: Yacc is a tool used to generate a parser, which is part of a compiler that processes structured input. Yacc takes a formal description of a language's grammar and produces a C program that parses that language

## **Bison**: Bison is a more modern version of Yacc that is often used in its place. Bison is compatible with Yacc and provides additional features

### SQLite Yacc/Bison Grammar File

In SQLite, the Yacc/Bison grammar file is typically named something like parse.y. This file contains a formal description of the SQL language grammar that SQLite understands. It specifies the structure of valid SQL statements and how these statements should be parsed into actions that the SQLite database engine can execute

### Key Concepts

1. **Tokens**: Basic symbols like keywords, identifiers, operators, and literals
2. **Grammar Rules**: Define how tokens combine to form valid statements
3. **Actions**: Code snippets are executed when a rule is matched

### Structure of a Yacc Grammar File

A typical Yacc grammar file has three sections

1. **Declarations**: Define tokens and C code includes
2. **Rules**: Grammar rules specifying how tokens form syntactic structures
3. **Auxiliary Functions**: Helper C functions are used in actions

### Yacc

## **1. Origins**: Yacc (Yet Another Compiler Compiler) is one of the earliest parser generators developed in the 1970s

## **2. Language**: Yacc uses syntax for specifying grammar rules and actions. It generates C code for the parser

## **3. Flexibility**: Yacc is powerful but somewhat rigid in its syntax and functionality. It's well-suited for traditional compiler construction tasks

## **4. Community**: Yacc has a large user base and is supported by various tools and resources

### Bison

## **1. Evolution**: Bison is a GNU project and an improved version of Yacc. It's backwards-compatible with Yacc but adds new features and improvements

## **2. Language**: Bison uses a syntax similar to Yacc for specifying grammar rules and actions. It generates C code for the parser

## **3. Features**: Bison offers enhancements over Yacc, such as better error reporting, support for additional programming languages, and improved performance

## **4. Open Source**: Being part of the GNU project, Bison is open source and actively maintained

### Lemon

## **1. Modern Approach**: Lemon is a more modern parser generator developed by the SQLite project

## **2. Simplicity**: Lemon aims to be simpler and easier to use than Yacc or Bison. It has a more straightforward syntax for specifying grammar rules and actions

## **3. Language**: Lemon uses its syntax, which is more human-readable and resembles pseudo-code. It generates C code for the parser

## **4. Designed for SQLite**: Lemon was specifically designed for use in the SQLite project, but it can be used for other projects as well

### Summary - Parser

- Yacc and Bison are older parser generators, while Lemon is a more modern alternative
- Yacc and Bison use similar syntax and are powerful but somewhat complex
- Lemon aims to be simpler and more user-friendly, with a syntax that's easier to understand
- Each tool has its strengths and is suitable for different types of projects and preferences

## Backend

The VDBE (Virtual Database Engine) is a core component of SQLite. It acts as a virtual machine within the SQLite architecture, executing the compiled bytecode that represents SQL statements. The VDBE is responsible for interpreting this bytecode and performing the corresponding database operations, such as selecting, inserting, updating, or deleting data

### Key Concepts of the VDBE

## **1. SQL Statement Execution**: When an SQL statement is executed in SQLite, it is first parsed into a parse tree by the SQL parser. The parse tree is then translated into a series of bytecode instructions by the VDBE. These instructions are a low-level, platform-independent representation of the SQL command

## **2. VDBE Bytecode**: The bytecode generated by the VDBE is similar to assembly language in a traditional CPU but is specific to SQLite. Each instruction in the bytecode corresponds to a specific operation, such as opening a table, reading a row, or computing a value

The VDBE includes a set of opcodes (operation codes) that define the different actions the engine can perform. Examples of opcodes include

- **OpenRead**: Open a table for reading
- **OpenWrite**: Open a table for writing
- **Column**: Extract a column value from a row
- **Insert**: Insert a new row into a table
- **Delete**: Delete a row from a table
- **Goto**: Jump to another instruction in the bytecode
- **Halt**: Stop the execution of the bytecode

## **3. Execution Flow**: The VDBE processes the bytecode instructions one by one. As it executes each instruction, it performs the corresponding database operation, manipulating tables, indexes, and data as specified by the SQL statement. The VDBE also handles control flow, such as loops and conditional branches, allowing complex SQL queries to be executed efficiently

## **4. Memory Management**: The VDBE uses a stack-based model for memory management during execution. It pushes intermediate results onto a stack and pops them off as needed for further operations. SQLite allocates and manages memory dynamically, ensuring that memory is used efficiently and freed when no longer needed

## **5. Error Handling**: If an error occurs during the execution of bytecode (e.g., a constraint violation or a failed disk operation), the VDBE can handle the error appropriately. It can roll back transactions, log errors, and return error messages to the user

## **6. Transaction Management**: The VDBE works closely with the SQLite transaction manager to ensure that database transactions are atomic, consistent, isolated, and durable (ACID). It generates the necessary bytecode instructions to begin, commit, or roll back transactions

## **7. Optimization**: The VDBE includes various optimizations to execute SQL queries as efficiently as possible. For example, it can optimize joins, make use of indexes, and eliminate unnecessary operations

## **8. Extensibility**: The design of the VDBE allows for extensibility. Developers can add custom functions, collations, or extensions to SQLite, and the VDBE can execute them as part of its bytecode

### Example Workflow of the VDBE

## **1. SQL Input**: The user inputs an SQL statement, such as SELECT * FROM users WHERE age > 30

## **2. Parsing and Bytecode Generation**: SQLite's SQL parser parses the statement and generates a parse tree. The VDBE compiles the parse tree into a sequence of bytecode instructions

## **3. Bytecode Execution**: The VDBE executes the bytecode instructions. For this query, it would open the users table, scan through the rows, filter rows where age > 30, and return the results to the application

## Key VDBE Opcodes for CREATE TABLE

## **1. Init**

- **Opcode**: `Init.
- **Description**: Initializes the VDBE. It sets up the environment for executing the bytecode instructions.
- **Usage in CREATE TABLE**: Sets up the environment to begin the table creation process.

## **2. OpenWrite**

- **Opcode**: `OpenWrite.
- **Description**: Opens a B-tree table for writing.
- **Usage in CREATE TABLE**: Opens the table to write the schema definition into the SQLite master table.

## **3. CreateTable**

- **Opcode**: `CreateTable.
- **Description**: Allocates a new table or index in the database.
- **Usage in CREATE TABLE**: Allocates a new table ID for the table being created.

## **4. String8**

- **Opcode**: `String8.
- **Description**: Pushes a string constant onto the VDBE stack.
- **Usage in CREATE TABLE**: Used to push the table name and column definitions onto the stack.

## **5. Integer**

- **Opcode**: `Integer.
- **Description**: Pushes an integer constant onto the VDBE stack.
- **Usage in CREATE TABLE**: Used to push integer values, such as the table ID.

## **6. MakeRecord**

- **Opcode**: `MakeRecord.
- **Description**: Creates a database record from values on the stack.
- **Usage in CREATE TABLE**: Combines table metadata (name, schema, etc.) into a record to be inserted into the SQLite master table.

## **7. Insert**

- **Opcode**: `Insert.
- **Description**: Insert a record into a table.
- **Usage in CREATE TABLE**: Inserts the newly created table's record into the SQLite master table.

## **8. Halt**

- **Opcode**: `Halt.
- **Description**: Stops the execution of the VDBE program.
- **Usage in CREATE TABLE**: Halts the VDBE program after the table creation process is complete.

### Detailed Flow for CREATE TABLE

## **1. Initialization**

- **Opcode**: `Init.
- **Function**: Sets up the VDBE environment.
- **Role**: Prepares for executing the CREATE TABLE statement.

## **2. Open Schema Table**

- **Opcode**: `OpenWrite.
- **Function**: Opens the SQLite master table (sqlite_master) for writing.
- **Role**: Allows writing the new table definition into the schema table.

## **3. Allocate Table ID**

- **Opcode**: `CreateTable.
- **Function**: Allocates a new table ID.
- **Role**: Reserves a new table ID for the table being created.

## **4. Prepare Column Definitions**

- **Opcode**: `String8.
- **Function**: Pushes the column definitions onto the stack.
- **Role**: Prepares the column information for the new table.

## **5. Form Table Record**

- **Opcode**: `MakeRecord.
- **Function**: Forms a database record from the table name, schema, and column definitions.
- **Role**: Creates a record that describes the new table.

## **6. Insert Table Record**

- **Opcode**: `Insert.
- **Function**: Inserts the new table record into the SQLite master table.
- **Role**: Commits the new table definition to the database schema.

## **7. Finalization**

- **Opcode**: `Halt.
- **Function**: Stops the VDBE program.
- **Role**: Ends the CREATE TABLE operation.

### Summary - VDBE

The VDBE is a critical part of SQLite's architecture, serving as the engine that executes SQL commands by interpreting and running bytecode instructions. Its role in managing query execution, transactions, and memory, along with its extensibility and optimizations, makes it essential for SQLite's efficiency and effectiveness as a lightweight, serverless database system.

## Pager

The Pager in SQLite is a key component responsible for managing the reading, writing, and caching of database pages between the database file on disk and memory. It plays a crucial role in ensuring data integrity, managing transactions, and handling concurrent access to the database.

### Key Responsibilities of the SQLite Pager

## **1. Page Management**: The Pager manages the movement of pages (fixed-size blocks of data, typically 4096 bytes) between the database file on disk and memory. Each page corresponds to a specific region of the database file. The Pager is responsible for reading pages into memory when they are needed and writing them back to disk when they are modified

## **2. Caching**: The Pager maintains a cache of recently accessed pages in memory to reduce the need for frequent disk I/O operations. This cache improves performance by allowing repeated access to the same data without re-reading it from disk. The size of the cache can be configured, allowing SQLite to be optimized for different environments (e.g., embedded systems with limited memory vs. desktop applications

## **3. Transaction Management**

- **Atomic Commit and Rollback**: The Pager ensures that transactions are atomic. If a transaction is committed, all changes are written to disk in a consistent state. If a transaction is rolled back, any changes made during the transaction are discarded, and the database is returned to its state before the transaction began.
- **Journal Files**: The Pager uses journal files to keep track of changes made during a transaction. These files record the original state of pages before they are modified. If a transaction is rolled back or if the database crashes, the journal can be used to restore the database to its original state.
  - **Rollback Journal**: The default journaling mode, where a separate journal file is created to store the original pages before modification.
  - **WAL (Write-Ahead Logging)**: An alternative journaling mode where changes are first written to a log before being applied to the database file. This allows for more efficient reads and better support for concurrent access.

## **4. Concurrency Control**: The Pager handles file locks to manage concurrent access to the database file. SQLite uses different locking levels (shared, reserved, pending, exclusive) to coordinate read and write access by multiple processes or threads. The Pager ensures that only one process can modify the database at a time while allowing multiple processes to read from the database concurrently

## **5. File I/O Abstraction**: The Pager abstracts the file I/O operations, providing a consistent interface for reading and writing pages to the database file. It manages the complexity of dealing with the file system, including handling partial writes, crashes, and ensuring durability. The Pager interacts with the Virtual File System (VFS) layer in SQLite, which handles the low-level details of file I/O, making SQLite portable across different operating systems

## **6. Crash Recovery**: In the event of a crash or unexpected shutdown, the Pager can use the journal files to recover the database to a consistent state. This recovery process involves either rolling back incomplete transactions or applying committed changes from the WAL file

## **7. Synchronization**: The Pager ensures that changes to the database file are synchronized to disk, making sure that data is not lost in the event of a power failure or crash. This is done through the use of the fsync() system call, which forces the operating system to flush buffered data to disk

### Example Workflow of the Pager

## **1. Reading a Page**: When the database engine needs to read data, it requests the specific page from the Pager. The Pager checks if the page is already in the cache. If it is, the page is returned immediately. If the page is not in the cache, the Pager reads it from the database file on disk and loads it into memory, adding it to the cache

## **2. Writing a Page**: When a page is modified, the Pager marks it as "dirty" in the cache. If the database is in a transaction, the original version of the page is saved in the rollback journal before the modification. Upon committing the transaction, the Pager writes the dirty pages back to the database file on disk and updates the journal to reflect that the changes have been committed

## **3. Committing a Transaction**: The Pager writes all modified (dirty) pages to disk. The Pager updates the journal to indicate that the transaction is complete. The journal file is either deleted (in rollback mode) or truncated/archived (in WAL mode

## **4. Rolling Back a Transaction**: The Pager discards any dirty pages from the cache. The original versions of the pages, stored in the journal, are restored to their pre-transaction state. The database file remains unchanged from its state before the transaction began

## Limitations

SQLite is a powerful, lightweight, and versatile database engine, but like any technology, it has certain limitations that make it more suitable for some applications than others.

### Concurrency and Locking

## **Limited Write Concurrency**: SQLite uses database-level locking, meaning that only one write operation can occur at a time. This can be a bottleneck in high-concurrency environments where multiple users or processes need to write to the database simultaneously

## **Reader-Writer Locking**: While multiple read operations can occur concurrently, as soon as a write operation begins, all other operations (including reads) must wait until the write is completed. This can lead to contention in scenarios with frequent reads and writes

### Database Size

## **Maximum Database Size**: SQLite databases are limited in size to 281 terabytes (2^47 bytes). While this is sufficient for most applications, it might be a limitation for very large-scale databases, such as those used by large enterprises or big data applications

## **File Size Limitations**: The maximum size of an individual SQLite database file is determined by the underlying filesystem, but SQLite itself imposes a limit of 140 terabytes per file. Additionally, certain older filesystems may have lower file size limits

### Performance Considerations

## **Heavy Write Operations**: Due to its single-writer model, SQLite can struggle with performance when there are many write operations. It's not well-suited for write-heavy workloads, especially those that require high throughput

## **Scaling Issues**: SQLite is not designed for large-scale, distributed systems. It doesn't support sharding or distributed transactions, which are often needed for applications requiring horizontal scaling across multiple servers

## **In-Memory Databases**: While SQLite supports in-memory databases, the data is lost when the application terminates. This makes it unsuitable for applications requiring persistence without a disk-based storage solution

### SQL Features

## **Missing Advanced Features**: SQLite does not support some advanced SQL features found in other RDBMS systems like MySQL, PostgreSQL, or Oracle, including

- **Stored Procedures**: SQLite does not support stored procedures or functions written in SQL.
- **Partial Indexes**: While SQLite supports basic indexes, it lacks support for some advanced indexing features like partial indexes or index-only scans.
- **Full Text Search (FTS)**: Although SQLite has FTS capabilities through extensions, it's not as comprehensive or performant as dedicated full-text search engines like Elasticsearch or PostgreSQL's full-text search.
- **Triggers and Views**: SQLite supports triggers and views, but with some limitations in complexity and functionality compared to more feature-rich databases.

### Data Types

## **Dynamic Typing**: SQLite uses dynamic typing, meaning that values can be stored in any column, regardless of its declared type. This flexibility can lead to data integrity issues if not carefully managed

## **Lack of Full Type Enforcement**: Unlike traditional RDBMS, SQLite does not enforce strict typing. For example, you can insert a string into a column declared as an integer, which can lead to inconsistent data

### Security Features

## **Limited Access Control**: SQLite does not support user accounts or roles natively. It lacks the sophisticated access control and user management features found in other database systems

## **Encryption**: While SQLite can be compiled with support for encryption (using the SQLite Encryption Extension or third-party libraries), it does not natively support encrypted databases out of the box. This can be a limitation for applications requiring robust data security

### Networked Access

## **No Native Network Support**: SQLite is a serverless database engine and does not provide network access by itself. It's designed to be embedded in applications, so if you need a database that can be accessed over a network, you'll need to implement your own solution or use a different database system

### Multi-User Applications

## **Single-User Focus**: SQLite is designed primarily for single-user applications. While it can be used in multi-user environments, it requires careful management to avoid contention and ensure data integrity, making it less ideal for applications with many simultaneous users

### Backup and Restore

## **Basic Backup Support**: While SQLite does support online backups via its VACUUM and backup API commands, it lacks the more advanced backup and restore features found in other databases, such as point-in-time recovery or incremental backups

### No Built-In Replication

## **Lack of Replication and High Availability Features**: SQLite does not have built-in support for replication, clustering, or high availability. If these features are required, other database systems like MySQL, PostgreSQL, or dedicated distributed databases are better choices

### Limited Support for Very Large Datasets

## **Indexing and Query Optimization**: While SQLite is efficient for small to medium datasets, it may not perform as well as other RDBMS systems when working with very large datasets, especially in terms of query optimization and indexing capabilities

### Concurrency with WAL Mode

## **Write-Ahead Logging (WAL) Limitations**: While WAL mode improves read concurrency and performance, it still has limitations, such as increased disk I/O and potential challenges with certain filesystem configurations

### Summary - Final

SQLite is a great choice for lightweight, embedded, or single-user applications where simplicity and minimal setup are key advantages. However, it has limitations in areas such as concurrency, large-scale data handling, advanced SQL features, and security, which make it less suitable for high-concurrency, enterprise-level, or highly secure applications. For such scenarios, a more robust RDBMS like MySQL, PostgreSQL, or a dedicated distributed database system may be more appropriate.
