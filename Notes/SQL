# SQL

## **Write Ahead Logs -**

**1.** WAL is a technique used to ensure that all modifications to the database are logged before they are applied.
2. This ensures that in the event of a crash, the database can be recovered to a consistent state by replaying the log entries.

## **How an Update is Written to WAL**

1. **Receive Write Request:**
    1. The database receives a write request, such as UPDATE table SET age = age + 1 WHERE age > 25.
2. **Generate Log Entry:**
    1. Before applying the update, the database generates a log entry that describes the changes. This log entry typically contains:
        1. The type of operation (e.g., UPDATE).
        2. The target table.
        3. The condition for selecting rows (e.g., age > 25).
        4. The changes to be made (e.g., age = age + 1).
        5. Any necessary metadata (e.g., transaction ID, timestamps).
3. **Write Log Entry to WAL:**
    1. The generated log entry is written to the WAL. This step ensures that the intended changes are safely stored on disk before they are applied to the database.
4. Apply Changes to Database:
    1. After the log entry is written to the WAL, the actual changes are applied to the database(i.e. written to the shared memory buffer(cache).
    In this case, rows where age > 25 are updated to increment the age value by 1.
5. Commit Transaction:
    1. Once the changes are applied, the transaction is committed. The commit record is also written to the WAL to mark the completion of the transaction.

**Write Request:**
When a write request comes, the database does two things synchronously:

1. Writes the transaction as it is to the write-ahead log file.
2. Finds the corresponding pages (which contain the required rows) from the disk(if not found in the shared pool), places the pages into the shared memory pool, and updates the pages.
3. After the WAL is written successfully and the pages are updated (also referred to as the dirty pages) in the memory pool, the transaction is marked successful and returns an OK response to the client.

**Read request:**

1. When a read request comes, the database tries to read the rows from the memory pool. If those pages are not found, it then tries to find the pages from the database on the disk, load them in the memory pool, and then return the required information.

### **Concept of “fsync” and “checkpointing”**

1. The process of periodically flushing the dirty pages from the shared memory pool to the disk is done using the “fsync” command.
2. Since we cannot keep writing infinitely to the WAL log files to maintain the records, whenever the dirty pages from the memory pool are persisted into the disk,the logs in the WAL are purged since we no longer need them. This process of purging the WAL records is called checkpointing.

https://vivekbansal.substack.com/p/database-internals-write-ahead-logging
