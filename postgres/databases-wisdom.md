### Why the fuck does a database even exist?
databases exist because data files sitting idle on the disk are highly vulnerable if the only thing managing and operating them is the OS
DBMS are softwares which sit between the OS and the data files, and do almost all the heavy lifting, act as a highly secure framework layer for data operations and management
Problems without a DB and how a DBMS solves it:

1. Crash Vulnerability: Writing a file, server crashes/system shuts down -> corrupted data
    what a DBMS does: internals such as WAL

2. Concurrency Collisions: Race conditions: inconsistent, corrupted data
    DB solution: MVCC



so far, you cant imagine an application without a proper DBMS.. a good DBMS gives priceless functionalities to applications to manage and operate with data. Robust internal architecture.. Database softwares handle them all under the hood from you to memory, process management to powerful engines serving applications on extremely huge datasets