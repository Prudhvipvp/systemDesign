Designing Data-Intensive Applications

***Reliability***

- The system should continue to work correctly (performing the correct function at the desired level of performance) even in the face of adversity (hardware or soft‐ ware faults, and even human error).

***Scalability***

- As the system grows (in data volume, traffic volume, or complexity), there should be reasonable ways of dealing with that growth.

***Maintainability***

- Over time, many different people will work on the system (engineering and operations, both maintaining current behavior and adapting the system to new use cases), and they should all be able to work on it productively.

* * *

**NoSql vs SQL -**

1.  In databases where you have many to 1 and many to many relationships between the relational sql tables - The corresponding noSql Document dbs(say Json as documents) will have very redundancy in their documents(/jsons) in them due to the nature of many to 1 and many to many. Also due to this redundancy, if there's an update, all the related document's data have to updated by your app code. If you need to prevent it, then a join needs to be done. Join for NoSql is to be done in application code.
2.  The main arguments in favour of the document data model(NoSql) are schema flexibility, better performance due to locality(no joins, all data in single doc/json), and that for some applications it is closer to the data structures used by the application(so no need to convert app's java object to a sql table's format). The relational model counters by providing better support for joins, and many-to-one and many-to-many relationships.  The locality advantage only applies if you need large parts of the document at the same time. The database typically needs to load the entire document, even if you access only a small portion of it, which can be wasteful on large documents. On updates to a document, the entire document usually needs to be rewritten—only modifications that don’t change the encoded size of a document can easily be per‐ formed in place \[19\]. For these reasons, it is generally recommended that you keep documents fairly small and avoid writes that increase the size of a document \[9\]. These performance limitations significantly reduce the set of situations in which document databases are useful.
3.  Note that, in MySQL since version 5.7, and IBM DB2 since version 10.5 \[30\] also have a similar level of support for JSON documents. We can have JSON as a column's datatype and even query based on that json's fields. In the same way, document DB's have started supporting relational type queries and joins across documents.

* * *

## Data Storage -

How are indexes stored?

- There are multiple ways, popular one is B-trees. Its a binary tree kind where at the root ,it has say 10 nodes and these 10 are doubly linked list. Now each node has value and pointer.
- Its like \*12 | 100, \*43 | 200, \*76 | 300 ........., this means all keys with value < 100, lie in the child thats at location \*12 and keys from <=100 to 200 are in location starting from 43....
- This way the height of tree if Log(n) and branching factor is how many branches each node has (generally 100s)
- So when there is a new key insertion, sometimes, you may need to split that one child Linkedlist to two and appropriately update the parent node.
- Thats why along with this Btree, a write Ahead Log is maintained, its just a log that maintains all the insert, update operations happening. So whenever db crashes and Btree is lost from RAM, WAL can be used to reconstruct it.

## ***OLTP VS OLAP (OL means Online)***

**<span style="color: rgb(100.000000%, 100.000000%, 100.000000%);">Transaction processing systems (OLTP)</span>**

1.  Small number of records per query, fetched by key Random-access, low-latency writes from user input End user/customer, via web application  
    Latest state of data (current point in time) ,Gigabytes to terabytes

**<span style="color: rgb(100.000000%, 100.000000%, 100.000000%);">Analytic systems (OLAP)</span>**

1.  Aggregate over large number of records Bulk import (ETL) or event stream  
    Internal analyst, for decision support History of events that happened over time Terabytes to petabytes

Since, the access patterns are very different for OLTP and OLAP, OLAP is used by analysts for aggregated queries. Although a same db used for OLTP can be used for OLAP also but may not be efficient at high scale OLAP. So, type of DBs used for both are different. And thats DATA WAREHOUSE come. These are just db where OLAP data is stored. The data comes from different OLTP DBs as shown - <img src="../_resources/Screenshot%202024-05-10%20at%205.37.10%20PM.png" alt="Screenshot 2024-05-10 at 5.37.10 PM.png" width="974" height="696" class="jop-noMdConv">

At flipkart, the fdp is the data warehouse.

- Many data warehouses are used in a fairly formulaic style, known as a star schema.
    
- At the center of the schema is a so-called fact table (in this example, it is called fact_sales). Each row of the fact table represents an event that occurred at a particular time (here, each row represents a customer’s purchase of a product)
    
- Some of the columns in the fact table are attributes, such as the price at which the product was sold and the cost of buying it from the supplier (allowing the profit mar‐ gin to be calculated). Other columns in the fact table are foreign key references to other tables, called dimension tables. As each row in the fact table represents an event, the dimensions represent the who, what, where, when, how, and why of the event.
    
- The name “star schema” comes from the fact that when the table relationships are visualized, the fact table is in the middle, surrounded by its dimension tables; the connections to these tables are like the rays of a star.
    
- So thats the logic of fact and dimension tables, fact is mostly event based, for every event(like product sold) there exists a row in fact and dimension table has all the meta data to which the fact table points via foreign key concepts.
    
- <img src="../_resources/Screenshot%202024-05-10%20at%205.45.35%20PM.png" alt="Screenshot 2024-05-10 at 5.45.35 PM.png" width="670" height="703" class="jop-noMdConv">

Also, the fact table can have billions of rows and 1000s of columns, But most analytical queries have where conditions in them and only need a few of the columns in return. So thats column oriented DBs are best suited for this. Basically, column DBs store all the data of a column together on disk , (rather than all row together).

<img src="../_resources/Screenshot%202024-05-10%20at%205.47.42%20PM.png" alt="Screenshot 2024-05-10 at 5.47.42 PM.png" width="553" height="581" class="jop-noMdConv">

- Also, column DBs are well very suited for compression becoz, mostly there is a same pattern in column values. i.e. although total rows is N , if you take distinct column values , it will be very less(see product_sk column above). So this feature helps in compressions. Compression Algos like BITMAP ENCODING will help here.
- Besides reducing the volume of data that needs to be loaded from disk in row DBs , column- oriented storage layouts are also good for making efficient use of CPU cycles. For example, the query engine can take a chunk of compressed column data that fits comfortably in the CPU’s L1 cache and iterate through it in a tight loop (that is, with no function calls). A CPU can execute such a loop much faster than code that requires a lot of function calls and conditions for each record that is processed.

**Materialised view vs view -**

In relational DBs and others, you can have this -

View -  its just a variable for a query. So you can say -

CREATE VIEW paidEmployee - select id,name from employees where salary > 0)

So now you can do query directly on paidEmployee as if its a table, but actually whenever you use view, the underlying query runs at the time and gives u latest results.

On contrary, Materialised View is the physicial result of a query thats stored in the memory/disk and(generally refresh every few hours etc). So you maintain it for frequently used queries for faster result(although it can be outdated by last refresh time).

## ==***Serialisation and Deserialisation -***==

- JSON and XML have good support for Unicode character strings (i.e., human- readable text), but they don’t support binary strings (sequences of bytes without a character encoding). Binary strings are a useful feature, so people get around this limitation by encoding the binary data as text using Base64(the same done in case of MLV7 Aerospike data write via json fre api). The schema is then used to indicate that the value should be interpreted as Base64-encoded. This works, but it’s somewhat hacky and increases the data size by 33%. Json serialisation doesn't do much to the data - its almost same as the json key-val pair. Just that whole json becomes stringified and its human readable - Now this string is passed over network (As bytes - each character 4 bytes)
- In both protoBuf and binary encoding formats, they reduce the size significantly and say there's a schema (with datatypes for each field) and in the data transmitted they just store the data's binary and what field number (say three fields in schema, each is give fieldTag - 1,2,3 and only this tag is stored in data part). Also, schema changes are easy and are both forward and backward compatible. Unlike json/xml, these binary encoding formats use multiple compression techniquies like variable len encoding etc to significantly reduce the data size. These extra optimisations of storing field names as ints OR not storing schema at all with data(Like in avro) are added optimisations. But majorly when compared to json, these serialise the data itself into non human readable binary encoded format
- AVRO is space wise most efficient and takes less space than protoBuff also becoz, avro also stores the schema first and data separately but in the data it just stores the field values but not which is the field. So the order in which schema fields are , same should be the order in the binary avro data also. Any mismatch will lead to data invalidation.
- So how does schema changes are handled? There will be writer's schema (generally for a table , the first row is the schema of the records in the table/ the version of schema and schema for that version is fetched from some db).And there exists a reader's schema that u should specify, now the avro readers automatically matches writer to reader schema by comparing field names(even if field name changes, field aliases are also checked). So ordering isnt a problem , also a new field added in reader's schema is fine(iff a default value is specified for this field in readers schema).
- Also, for dynamically generated schemas  = avro is better, say where a big sql table has fields and later 2 fields are(1 added , 1 deleted) and you are reading it. For avro , its very easy by above writer-reader schema . But for protobuf etc, since they store fieldTags also, whenever columns are added/deleted, the admin should carefully assign new fieldTags to new columns(making sure no tag is re used).
- Generally ,dbs have their process that does this serialisation and de-ser of data into required formats.

* * *

## *==**REPLICATION -**==*

- **Master-slave replication -**
    
- In this a single master/leader receives writes and reads and those are replicated to all followers which serve only reads. Now is it sync/async? generally its sync for one node and async for rest of nodes meaning that leader just waits till one of the follower acked the write and responds to the user that write is successfull while async writing is gng on to rest of the followers.
    
- How is a follower crash recovered? In its local disk, follower maintains a log of all writes from its leader, so once crash happens, it can recover from this log and then ask the leader all the writes post the crash time.
    
- **Failover** - It means whenever leader itself crashes, the rest nodes decide a leader(by consensus/voting/etc) and this selection of new leader is called failover. After this if the old leader comes up, it has to be notified that its no longer leader. Or else 'split brain' problem arises, which is when 2 leaders exist and both of them accept writes. This can lead to data inconsistency
    
- Now for replication Logs the popular 2 types -
    
    - Write Ahead Logs(WAL) - In this logs are stored about what bytes are changed in the disk level. This is problematic becoz it is coupled with sotrage engine, if that changes/ software update/ leader and follower has different software versions- In all such cases, the replication log cant be used to restore data.
    - Logical(Row based) Log replication - In this the actual data is logged, if its insert, the whole row is logged. If delete, the primary key and if Update - the updated columns with prim key. This way its decoupled from storage engine.
- There is also TRIGGER BASED REPLICATION - This is replication triggered by application code. In this we decide when to only replicate, which subset of data to replicate etc. In sql, we have 'triggers'. A trigger lets you register custom application code that is automatically executed when a data change (write transaction) occurs in a database system.
    
- REPLICATION LAGS -
    
    - Lag is the lag between leader write and the last followers write. (If its not sync replication)
    - One of the problems -
        1.  READ AFTER WRITE - due to this is user after updating his data, not seeing once he reloads the page(this read is maybe served by a async replica). For such cases, we need to enforce 'read after write consistency' which means reads made by user. This is a guarantee that if the user reloads the page, they will always see any updates they submitted themselves. It makes no promises about other users: other users’ updates may not be visible until some later time. How to implement it are many ways - One is for a minute after user write, re route all the read req by that same user to leader only. etc
        2.  MONOTONIC READS - Other problem due to lag is - going backward in time. Say user is reloading constantly, and he first got served from replica 1 and then from replica 2(for the same page) and if replica 2 is lagging behind replica1. Then user is essentially seeing older data. To prevent this, monotonic reads consistency needs to be enforced. monotonic reads only means that if one user makes several reads in sequence, they will not see time go backward— i.e., they will not read older data after having previously read newer data. Its achieved by say - a user's request always goes to a same replica.
        3.  CONSISTENT PREFIX READS - There can be some writes that are dependent and causal. I.e. something like say question and an answer. In the same order. Now if both lie in separate replicas with diff lags. A user may see the answer first before the question. So to prevent this - consistent prefix reads. This guarantee says that if a sequence of writes happens in a certain order, then anyone reading those writes will see them appear in the same order. This is a particular probelm in sharded Dbs since diff data goes to diff shards.(If we enforce causally related data should go to same shard that may be solvable).
- MULTI LEADER REPLICATION -
    
    - IF you have databases across geo location datacentres, then having a single leader will be latent due to cross DC lags and writes redirected to a different DC(since leader is in diff DC).
    - Such cases, multi leaders are used. Its the same concept, leaders accepts R/W and replicate(now to other leaders and followers also). Generally, its like a leader for each DC. Issue with this is as said above, write conflicts. A user can edit a wiki page's title from A TO C (that goes to DC 1 leader) and other user edited it from A TO B(that went to DC2 leader). This is a conflict.
    - So conflict resolving algos should be there either at application level code/ DB level(something like enforcing the highest timestamp as final write value etc but its not reliable can lead to dataloss)
    - So mostly the conflicts are resolved custom in app code. The db triggers whenever it sees a conflict and all those conflicting writes go to application(when a subsequent read is made) that decides whats the final write value to be.
    - The causal problem can happen here among leaders also. Say leader1 got an insert row request and leader2 got this replication log and updated in it, post this update request of the same row is received by leader2. Theres also a leader3, now for some reason, the replication log it received is first update request from leader2 and then the insert request from leader1. Which is causally incorrect. Simply depending on timestamp isn't correct becoz clocks also cant be trusted to be perfectly in sync. Version Vectors is a solution for this problem.
- LEADERLESS REPLICATION:
    
    - In this (like cassandra) there is no leader, all r/w are sent to all replicas parallely and they the write/read ack is given by a subset of nodes.
    - This is called quorum.Say you have n nodes and w and r as the write and read count of replicas that should atleast acknowledge your w/r. It should be w+r > n for avoiding stale data. Say n=3(replication factor = 3) and you gave write request to all 3 and w =2 so if any 2 nodes ack the write, the client gets 200. Now if the read is made, which is also made for all nodes and response is waited till we get from atleast r nodes, in this case r=2, so atleast there is 1 node with latest data, Now based on version number of the data, we can decide which is latest and return that to client.
    - The rule is even though w+r > n, if the writes to nodes which acked(the w) and the later read response given by r nodes, if there is no overlap between them, then stale data would be read. Thats why we choose generally, w=r=(n+1)//2. So that overlap must exist.
    - How are the stale replicas catch up -  there are diff techniques(pull/push based) -
        - Read repair - In this when a read is done and from the r quorum, if any node gives stale data, at that point the data is updated. So if its a infrequent read, then stale data can be present for a long time.
        - ANTI ENTROPY - In this a background process, keeps check of any diff between data and copies it.
    - Although you maintain a good quourm with w,r = n+1//2 , there can be edge cases, where you read old data . So the checks must verify it strictly
- SLOPPY QUORUM -
    
    - We saw the concept of w,r, and n. Now  say replication factor is 3. (n=3). Now the actual number of db nodes can be many say 7. But n=3 and w=r=3. Now, say you got a write request, and they go to the 3 designated nodes(say decided by consistent hashing). Say, out of the 3 nodes, 1 is down for some reason and hence quorum cant be reached, In such case, the write is done to another node(among the total 7 , we send it to any other node which is not part of the 3 nodes decided by consistent hashing). Now this new neighbor node accepts write and write 200 is given, later when the damaged home node is up, the data is transferred from neighbor to home.(This transfer is called hinted handoff) and this process of quorum transfer to neighbor nodes is called sloppy quorum. <span style="color: #ffffff;">Thus, even under partial temporary failures, the system can meet high availability agreements.</span>
    - Sloppy quorums are particularly useful for increasing write availability: as long as any w nodes are available(even outside the home n replica nodes), the database can accept writes. However, this means that even when w + r > n, you cannot be sure to read the latest value for a key, because the latest value may have been temporarily written to some nodes outside of n.
- CONCURRENT WRITES -
    
    - Say n=3 and you got 2 concurrent writes set x=A and set x=B. Now for nodes 1,2 set X=B reached last and for node3, set X=A reached last. Now if overwrite is allowed. Then when we read, node 1 gives B and node 2 gives A. and they can never reach a consistency ever.
    - So conflict resolution of concurrent writes must be explicitly done. One popular is last write wins(LWW). In this simply, the timestamp of write which is latest is considered final value and thats returned and also updated in the other node.
    - LWW achieves the goal of eventual convergence, but at the cost of durability. All the writes before are lost. The only safe way of using a database with LWW is to ensure that a key is only written once. Cassandra uses LWW and if we ensure for every write, the write key is a UUID. So each write will then be unique.
- MERGING CONCURRENT WRITTEN VALUES - How to ensure, data isn't lost like LWW -  Versioning is used in such cases. **Version Vectors** is an algo for this which keeps track of the whether a write is an overwrite/concurrent write(i.e. an algorithm to determine whether one operation happened before another(an overwrite), or whether they happened concurrently) and takes call accordingly. The data is not lost but both copies are kept. The logic is client when writes , the server returns 200 along with a version number. Client should send back this version num for every consequent write. Now if  two clients are concurrently writing, they may send different version numbers and based on which is greater, it can decide they are concurrent writes. Similarly , if the same client is writing again, it sends the same version number - so meaning its an overwrite and not a concurrent write. So we can overwrite the data and no need to keep both copies(in case of concurrent, the db will keep both the write values with it and give a new version number to it)
    
- Single-leader replication is popular because it is fairly easy to understand and there is no conflict resolution to worry about(not much issue for concurrent writes). Multi-leader and leaderless replication can be more robust in the presence of faulty nodes, network interruptions, and latency spikes—at the cost of being harder to reason about and providing only very weak consistency guarantees - because they allow multiple writes to happen concurrently, conflicts may occur.
    

* * *

## *==**PARTITIONING** -==*

- ![Screenshot 2024-05-12 at 10.22.35 PM.png](../_resources/Screenshot%202024-05-12%20at%2010.22.35%20PM.png)

&nbsp;So, the above replication with partitioning. Note that each node will have a leader partition and the followers will be other nodes(for fault tolerance).

- we can keep keys in sorted order with in a partition. This has the advantage that range scans are easy, say key is timestamp. now you can do a easy scan for a month of data, since all data is sorted, so disk access/ memory access is easier since adjacent data)

==**HASHING BASED PARTITIONING -**==

- [ ] Simply hashing the key and assigning the partition based on the hash value i.e. say partiont1 contains hash values from 0 to 2999 etc..
    
- [ ] This will give range to uniform distribution and can avoid hotspotting(in most cases), but scans can be tough, becoz similar data is now randomly partitioned across different nodes. One better way is having key as a mix like userID+timestamp - now all userID data will lie in single node within each its sorted on timestamp.
    
- [ ] HOTSPOT - For celebrity users, still hots potting can happen - To prevent this - For example, if one key is known to be very hot, a simple technique is to add a random number to the beginning or end of the key. Just a two-digit decimal random number would split the writes to the key evenly across 100 different keys, allowing those keys to be distributed to different partitions. But issue is for every read, now these 100 different key's partitions have to be searched(Some optimal and clever way of adding salt can be done instead of a rand number).
    
- [ ] **Primary index:** A primary index is an index on a set of fields that includes the unique primary key for the field and is guaranteed not to contain duplicates. Also Called a **Clustered index**. eg. Employee ID can be Example of it.
    
- [ ] **Secondary index:** A Secondary index is an index that is not a primary index and may have duplicates. eg. Employee name can be example of it. Because Employee name can have similar values.
    

==**PARTITIONING WITH SECONDARY INDEX -**==

1.  Say the document is {id:"","color":"","make":"Honda"}, ..........
2.  Now the prim key/index is id , so you partition by id. But while querying u can query based on color also, give all docs of red color. Now for this purpose of secondary index queries we do indexing(in sql and nosql also). So here also we can do indexing- There are 2 types -

### ==Document based index -==

1.  In this index is created within each partition for all docs present in that partition. This way, whenever u want all red docs - you need to query all partitions, becoz red can be anywhere. This approach is scatter/gather and used by mongoDB and other famous DBs, The writes are faster as index is within that partition only but reads are slower due to parallel multi partitions read.
2.  So this secondary index is called local index. Becoz each index is local to that partition

![Screenshot 2024-05-14 at 12.22.32 PM.png](../_resources/Screenshot%202024-05-14%20at%2012.22.32%20PM.png)

==Term based Index -==

1.  In opposition to above approach, here we create a global index. Here also same index is created on color within each partitions but after that, that indexes are further partitioned i.e. all colors starting from a to h lie in partition1 and h to z lie in partition2. This way we make sure a query for a color goes to a single partition. But downside , is the index creation process needs to communicate with all partition nodes. Say a new entry is added to partition1(based on its unique doc id) and now if its color is violet, it should go to partition2. Similarly, if theres a index on make also, then again same thing. So the writes are slower and also we expect index creation to be done asap after write. But in this case , that may not be possible due to network comms.
2.  Also, most dbs employ async index creation and not in sync with the writes.
3.  Here the word 'term' means the field in a doc.

![Screenshot 2024-05-14 at 12.27.02 PM.png](../_resources/Screenshot%202024-05-14%20at%2012.27.02%20PM.png)

### ==**Rebalancing partitions -**==

1.  Its known that we should not assign partitons simply by hash(Key)% N. becoz this will involve great rebalancing if a node is added/removed. Better way can be that it’s best to divide the possible hashes into ranges and assign each range to a partition (e.g., assign key to partition 0 if 0 ≤ hash(key) < b0, to partition 1 if b0 ≤ hash(key) < b1, etc.).
2.  There's also dynamic partitioning based on more keys added/removed from a partition, if they exceed a said limit. It breaks into two(/merged into 1).

&nbsp;

***How are requests routed to correct partition ? i.e. SERVICE DISCOVERY***

- Most use zookeeper for this which keeps track of all nodes and what partitions lie where.

![Screenshot 2024-05-14 at 12.39.25 PM.png](../_resources/Screenshot%202024-05-14%20at%2012.39.25%20PM.png)

Some dbs like cassandra use gossip protocol where every node is aware of where partitions lie in which node. so client calls any node and that node redirects it to correct node.

* * *

## ==**TRANSACTIONS**== ==ISOLATION LEVELS==

Not all dbs need Serialisable isolation level which means two txns run concurrently , they behave as if both have run sequentially.

**Dirty Read** - Reading a value that another transaction has written but not yet commited/aborted. Same thing for dirty Write.

Other isolation levels-

==**1\. Read commited -**==

1.  It says, you can read and write values only that are committed(i.e. dirty reads arent possible).
2.  Most databases prevent dirty reads using the approach illustrated in for every object that is written, the database remembers both the old committed value and the new value set by the transaction that currently holds the write lock. While the transaction is ongoing, any other transactions that read the object are simply given the old value.

**==2\. Snapshot Isolation/Repeatable Reads -==**

1.  Even with read committed, theres a problem, in long running read queries that can involve two different entities like say bank transfer amount from accA(500 $) to accB(500$) , 100$ are to be transferred. In this, even with read committed. Lets say there's a transaction thats running to transfer this amount.
2.  At the same time, you try to read account balance for both A and B. A has done a read operation, when the transfer process has started, so A amount has already been debited by 100$ and its committed. So A read will see 400(since its committed value). But it being some long running query, the B balance read query, has started before the transfer to B has committed and so, B will read the last committed value which is 500$. So total 900$, which should have been 1000. Although doing a refresh of read again, may reflect correct values. This issues happens when we try to copy databases, then some parts may be older and some new.
3.  This problem is called 'READ SKEWS/ Non repeatable reads"
4.  For that we have snapshot isolation - mainly used for long running read queries. It basically maintains snapshots of data at different points in time. And whenever a read transaction starts, it keep reading from the snapshot of db thats present at start of txn and it remains that only till txn commits. So , no intermediate values possible.
5.  <span style="color: #ffffff;">The “Repeatable Read” is the default isolation level set in MySQL.</span>

## **LOST UPDATES =**

1.  When two concurrency writes happen which read and write the read+1 value, Then one of the write can be lost, becoz it has read the older value. For this some Dbs give atomic functionalities. Atomic operations are usually implemented by taking an exclusive lock on the object when it is read so that no other transaction can read it until the update has been applied.
2.  Automatic lost update detection  -  Atomic operations and locks are ways of preventing lost updates by forcing the read- modify-write cycles to happen sequentially. An alternative is to allow them to execute in parallel and, if the transaction manager detects a lost update, abort the transaction and force it to retry its read-modify-write cycle.

==WRITE SKEWS -==

Say a use case of claiming a user name thats un used. Two users input the same name concurrently. Now the 2 individual txns run that do -

1.  Check if current user name is in DB?  =  Both of them return true
2.  Insert the user name in db -> So due to this both got same name(although this can be prevented is username is prim key and so second write wont happen). The point is this kind of problem is write skew. Clearly, if both the txns werent concurrent, the result would have been different. To solve this, we again need to use atomic - by read locking the whole rows in table(since the is present query does run on all rows -  This approach is called materializing conflicts and is hard and error prone and better to use serialisable isolation instead of this.

So, there are many such examples - the pattern is -

1.  First you read and get a result.
2.  Based on the read result, you take a decision to write.
3.  Now post this, if you read again, the read result gives a different result because the write changed the set of rows matching the search condition.

This effect, where a write in one transaction changes the result of a search query in another transaction, is called a **phantom**. Snapshot isolation avoids phantoms in read-only queries, but in read-write transactions like the examples we discussed, phantoms can lead to particularly tricky cases of write skew. So, use serialisable isolation as in other words, using this the database prevents all possible race conditions.

## **How is serialisable implemented -**

*==**1\. Serial execution -**==*

Here basically, the things are run in a single thread. So, by default, no two process can read a shared resource at same time( you can ignore the interleaving of process's due to OS scheduling as this generally doesn't happen becoz operations happen/can be made to happen atomically on single thread).

The conditions for using this -

- Write throughput must be low enough to be handled on a single CPU core, or else transactions need to be partitioned without requiring cross-partition coordination.
- If you can make each transaction very fast to execute, and the transaction throughput is low enough to process on a single CPU core,

Stored Procedures - In cases, where a transactions needs to be spanned across multiple http requests from client , we cant wait for client to call and its very inefficient. Stored procedures are used in such places . Instead of interactive txns. In below the query processor is some db query layer thats near to db.![Screenshot 2024-05-16 at 5.37.17 PM.png](../_resources/Screenshot%202024-05-16%20at%205.37.17%20PM.png)

So, clearly stored procedures are much better in such cases.

### *==**2\. 2PL - 2 phase locking -**==*

1.  This is one of the famous serialisable implementation to prevent race. It says -
2.  **Writers block both writers and readers. Readers block only writers.** So multi reads can happen.(Note that Snapshot isolation has the mantra readers never block writers, and writers never block readers)
3.  How its done - Using shared and exclusive locks - (shared lock means other shared locks also can use that lock but not the excluse ones, while exclusive locks means only the current one can access). So readers will use shared lock and writers use exclusive lock. Since so many locks are in use, it can happen quite easily that transaction A is stuck waiting for transaction B to release its lock, and vice versa. This situation is called deadlock. The database automatically detects deadlocks between transactions and aborts one of them so that the others can make progress.
4.  The big downside of two-phase locking, and the reason why it hasn’t been used by everybody since the 1970s, is performance: transaction throughput and response times of queries are significantly worse under two-phase locking than under weak isolation.

### **==*3\. Serialisable snapshot Isolation (SSI) -*==**

1.  This is considered best performing algo for serialisable isolation. It is basically snapshot isolation . But on top of it,it has a detector that checks after a txn committed, did any serialisable condition break? if yes abort that transaction.  Its better performant becoz of faster reads, in particular, read-only queries can run on a consistent snapshot without requiring any locks, which is very appealing for read-heavy workloads. But in cases of very high concurrent writes, this may be bad, cause that will cause many aborts (which is a overhead). **Also here writers dont block reads**(since snapshot isolation).
2.  Basically, SSI is an optimistic approach for solving serialisability meaning allowing transactions to proceed without blocking. It assumes nothing will go wrong. While the other two(2PL and serial execution) are pessimisstic meaning they assume each transaction can break serialisability, so acquire locks to prevent this.

* * *

## ==***THE TROUBLE WITH DISTRIBUTED SYSTEMS -  WHAT ALL COULD GO WRONG?***==

- The one of the major problem is network delays that happen among nodes. We use IP/TCP/UDP for internet packets transfer and by design we cant guarantee how much time a message may take for a transfer from a node to node. The receiver node may be down/ packet got lost while transmission anything can happen.
    
- Thats why we have timeouts configured, instead of waiting for the response indefinitely.
    
- How are phone calls so reliable unlike internet connections?
    
    - A circuit in a telephone network is very different from a TCP connection: a circuit is a fixed amount of reserved bandwidth which nobody else can use while the circuit is established, whereas the packets of a TCP connection opportunistically use whatever network bandwidth is available.
        
    - Saya telephone connection wire has carry 10K simulataneous calls. And for phone calls this is static, i.e. even you are the single user making phone call . Or there are 10K people calling each other. Each of them gets the same bandwidth which is pre defined. Due to this fixed bandwidth allocation, the time taken for communication is reliable.
        
    - Unlike in internet, for better hardware resource utilisation , we dont do fixed bandwidth allocation and it is dynamic
        
    - The internet shares network bandwidth dynamically. Senders push and jostle with each other to get their packets over the wire as quickly as possible, and the network switches decide which packet to send(Like a OS scheduling) (i.e., the bandwidth allocation) from one moment to the next. This approach has the downside of queueing, but the advantage is that it maximizes utilization of the wire. The wire has a fixed cost, so if you utilize it better, each byte you send over the wire is cheaper.
        
    - So, Variable delays in networks are not a law of nature, but simply the result of a cost/ benefit trade-off.
        
- Another major unseen issue is "TIME CLOCKS"
    
- In a distributed node setup, each node has its own 'time of the day' clock which is basically java.getSYstemTime() telling the epoch seconds from 1970...
    
- But the clocks across machines cant be very precisely synchronous. There exists a delay of 100ms which is normal. Due to this, we cant rely on algo like Last Write Wins , where if both A and B write at a node, and whichever is last timestamp wins. Although A is first and then B incremented A. If the clock of the client and the receiving node isnt in sync , then A may overwrite B making B write lost.
    
- So, If you use software that requires synchronized clocks, it is essential that you also carefully monitor the clock offsets between all the machines. Any node whose clock drifts too far from the others should be declared dead and removed from the cluster. Such monitoring ensures that you notice the broken clocks before they can cause too much damage.
    
- Thats why like said in Snapshot Isolation algo's  versioning vectors are used in such cases, where we cant rely on timestamp as an identifier of causality.
    
- An emerging idea is to treat GC pauses like brief planned outages of a node, and to let other nodes handle requests from clients while one node is collecting its garbage. If the runtime can warn the application that a node soon requires a GC pause, the application can stop sending new requests to that node, wait for it to finish process‐ ing outstanding requests, and then perform the GC while no requests are in progress. This trick hides GC pauses from clients and reduces the high percentiles of response time \[70, 71\]. Some latency-sensitive financial trading systems \[72\] use this approach.
    

## ==**LINEARIZABILITY - CONSENSUS:**==

1.  What does it mean , a distributed db is linearisable - It means for a client it should feel like a db has a single replica (although there may be multiple nodes and multiple replicas).
2.  Why? - This is required for the inconsistencies that happen due to replication lag, although write is success, some clients may see one older read value, and others the latest value.
3.  **Linearisability** - Its basically a recency guarantee - When multiple clients are asking for reads and writes. It doesnt matter which client performed which operation first(becoz due to network delays all may not reach the servers in same order and also response cant be in same order). It says, once any of the client read a value 'R' , all the next reads from different clients should also be 'R'. Here we are assuming 'R' was the latest value for that key. IF there comes a write request for the key, and concurrently other read request - it may be possible that read request returned the new written value even before write has responded to client with 'ok' (due to N/W delays or whatever). But once a client got this new written value, all the further clients should get this new value only.

![Screenshot 2024-05-18 at 5.04.55 PM.png](../_resources/Screenshot%202024-05-18%20at%205.04.55%20PM.png)

NOTE: A database may provide both serializability and linearizability, and this combination is known as strict serializability or strong one-copy serializability (strong-1SR) \[4, 13\]. Implementations of serializability based on two-phase locking or actual serial execution  are typically linearizable.However, serializable snapshot isolationis not linearizable: by design, it makes reads from a consistent snapshot, to avoid lock contention between readers and writers. The whole point of a consistent snapshot is that it does not include writes that are more recent than the snapshot, and thus reads from the snapshot are not linearizable.

When its used?

1.  In leader election - One way of electing a leader is to use a lock: every node that starts up tries to acquire the lock, and the one that succeeds becomes the leader \[14\]. No matter how this lock is implemented, it must be linearizable: all nodes must agree which node owns the lock; otherwise it is useless.
2.  InUniqueness constraints are common in databases: for example, a username or email address must uniquely identify one user, and in a file storage service there cannot be two files with the same path and filename
3.  As in below example, of ot pccurs that webserver sent the photoId to queue and queue consumption is quite fast that the image resizer tried to fetch the photoID even before its replicated. So in such cases, linearisability is a requirement. This problem arises because there are two different communication channels between the web server and the resizer:
4.  ![Screenshot 2024-05-18 at 5.12.46 PM.png](../_resources/Screenshot%202024-05-18%20at%205.12.46%20PM.png)

**CAP THEOREM -**

- So the Linearisability we talked is the same concept as 'consistency' in CAP. Having a consistent reads across clients.
    
- CAP is not choose any 2 out of 3. Becoz network partition will anyways happen if u like it or not. So it should rather be -
    
- At times when the network is working correctly, a system can provide both consis‐ tency (linearizability) and total availability. When a network fault occurs, you have to choose between either linearizability or total availability. Thus, a better way of phrasing **CAP would be either Consistent or Available when Partitioned**
    
- **PROOF -**
    
    - - If your application requires linearizability, and some replicas are disconnected from the other replicas due to a network problem, then some replicas cannot process requests while they are disconnected: they must either wait until the network problem is fixed, or return an error because its a linearisable system and cant return a stale value that it contains (It should do the replication from the other network of replicas before returning result but due to n/w error that doesnt happen) (either way, they become unavailable).
            
            ``````
            `````
            ````
            ```
            - If your application does not require linearizability, then it can be written in a way that each replica can process requests independently, even if it is disconnected from other replicas (e.g., multi-leader replication). In this case, the application can remain available in the face of a network problem, but its behavior is not linearizable.
            ```
            ````
            `````
            ``````
            
- If you want linearizability, the response time of read and write requests is at least proportional to the uncertainty of delays in the network. In a network with highly variable delays, like most computer networks , the response time of linearizable reads and writes is inevitably going to be high. A faster algorithm for linearizability does not exist, but weaker consistency models can be much faster, so this trade-off is important for latency-sensitive systems.
    

**DISTRIBUTED ATOMIC TRANSACTIONS ALGORITHMS -**

- A transaction commit must be irrevocable—you are not allowed to change your mind and retroactively abort a transaction after it has been committed. The reason for this rule is that once data has been committed, it becomes visible to other transac‐ tions, and thus other clients may start relying on that data; this principle forms the basis of read committed isolation. If a transaction was allowed to abort after committing, any transactions that read the committed data would be based on data that was retroactively declared not to have existed—so they would have to be reverted as well.

1\. TWO PHASE COMMIT(2PC) -

- Two-phase commit is an algorithm for achieving atomic transaction commit across multiple nodes—i.e., to ensure that either all nodes commit or all nodes abort. It is a classic algorithm in distributed databases
    
- 2PC provides atomic commit in a distributed database, whereas 2PL provides serializable isolation. Both are completely different concepts.
    
- ![Screenshot 2024-05-19 at 3.36.46 PM.png](../_resources/Screenshot%202024-05-19%20at%203.36.46%20PM.png)
    
- If any of the nodes, reply no in prepare phase(This phase is just asking node if u r ready to commit) - then whole commit isnt done on any node.
    
    - See that there's a new component required in 2PC, which is the coordinator
- Once ==coordinator== gets yes for the prepare from all nodes. It writes a commit gng to happen in its log. And this means the commit WILL happen definitely. And then commit request is sent to each nodes. If any of them is crashes/timedout. The coordinator will retry forever and whenever the nodes comes up, the commit will be made(either coordinator has retried/ the node itself maintains a global txn id which it will ask to coordinator, is this txn Id commit/abort). Becoz the node has promised in prepare phase.
    
- What if coordinator crashes?
    
    - If it crashes, before the prepare requests, then nodes can themselves abort(i.e. if coordinator doesnt send any prepare request).
    - But if it crashes, after the prepare request, then nodes have to must wait for coordinator decision to commit/abort. It cant take a call itself, becoz may be some of the other nodes have committed/aborted. And nodes wont talk among themselves, they only talk to coord.
    - So in such cases, nodes will indefinitely wait for coordinator to come up. The only way 2PC can complete is by waiting for the coordinator to recover. This is why the coordinator must write its commit or abort decision to a transaction log on disk before sending commit or abort requests to participants: when the coordinator recovers, it determines the status of all in-doubt transactions(the ones which are waiting after prepare phase) by reading its transaction log. Any transactions that don’t have a commit record in the coordinator’s log are aborted.
    - If the coor‐ dinator has crashed and takes 20 minutes to start up again, those locks it has put on the DB's rows to commit/abort will be held for 20 minutes. If the coordinator’s log is entirely lost for some reason, those locks will be held forever—or at least until the situation is manually resolved by an administrator.
- Distributed transactions, especially those implemented with two-phase commit, have a mixed reputation. On the one hand, they are seen as providing an important safety guarantee that would be hard to achieve otherwise; on the other hand, they are criticized for causing operational problems, killing performance, and promising more than they can deliver
    

==**Heterogeneous distributed transactions -**==

- Even in heterogenous distributed systems, that is one db is sql, other is kafka etc.. The atomic transactional can be achieved(although difficult).
- X/Open XA (short for eXtended Architecture) is a standard for implementing two- phase commit across heterogeneous technologies. It basically is a 'C' api for interfacing with a transaction coordinator. XA assumes that your application uses a network driver or client library to communicate with the participant databases or messaging services. So using the api calls probably, the coordinator will ask driver and it will talk to participant DBs.

&nbsp;**Epoch numbering for leader election -**

- We saw that we cant have split brain i,..e 2 or more leaders, as due to this both will accept writes and no sync and finally the data keeps diverging and all nodes wont be in Sync.
    
- In single leader  also, a strict algo to decide a leader whenever a leader fails is required and if that old leader comes back, it must know that its no more a leader. A simple algo is using epoch numbering -
    
- Every time the current leader is thought to be dead, a vote is started among the nodes to elect a new leader. This election is given an incremented epoch number, and thus epoch numbers are totally ordered and monotonically increasing. If there is a conflict between two different leaders in two different epochs (perhaps because the previous leader actually wasn’t dead after all), then the leader with the higher epoch number prevails. Before a leader is allowed to decide anything, it must first check that there isn’t some other leader with a higher epoch number which might take a conflicting decision. So for this, before taking any decision, even the leader will ask all its nodes, and if any of the nodes detects that there is an other leader with higher epoch, then that decision wont be taken and that old leader can now come down.
    
- Tools like ZooKeeper play an important role in providing an “outsourced” consensus, failure detection, and membership service that applications can use.
    

* * *

## ==**BATCH PROCESSING -**==

MAP REDUCE -

1.  The distributed data (on say HDFS) is first read and breakup into records(can think each row is a record).
    
2.  Mapper -
    
    - The mapper is called once for every input record, and its job is to extract the key and value from the input record. For each input, it may generate any number of key-value pairs (including none). It does not keep any state from one input record to the next, so each record is handled independently. In some example, key can be the URL
3.  Reducer
    
    - The MapReduce framework takes the key-value pairs produced by the mappers, collects all the values belonging to the same key, and calls the reducer with an iterator over that collection of values. The reducer can produce output records (such as the number of occurrences of the same URL).
4.  Basically, in hdfs world, where say a file is split into 4 partitions over 4 datanodes. NameNodes will have metadata of which data in which partition. Now the mapper function will run. Num of mapper functions = num of input splits = in this case 4. Each mapper function will run in the partition which it will be reading, so no data transfer happens, just the mapper function itself is transferred to corresponding machines. So 4 outputs will be produced(1 per partition) But user generally want 1 single output for whole data. This is done by reduce function, which combines data over the 4 partitions does something and gives an output.
    
5.  In MapReduce job, per job it can have a mpper and a reducer only. So most workflows need user to make this chain connection of job1 to job2 to job3. (Unlike spark, where it automatically breaks it down into stages and does it for you by creating the DAG)
    
6.  One way of looking at this architecture is that mappers “send messages” to the reducers. When a mapper emits a key-value pair, the key acts like the destination address to which the value should be delivered. Even though the key is just an arbitrary string (not an actual network address like an IP address and port number), it behaves like an address: all key-value pairs with the same key will be delivered to the same destination (a call to the reducer).
    
7.  Also, using map reduce job to write to some DB is not a great option. map reduce is highly parallel, so DB may choke. And MapReduce provides a clean all-or-nothing guarantee for job output: if a job succeeds, the result is the output of running every task exactly once, even if some tasks failed and had to be retried along the way; if the entire job fails, no output is produced. However, writing to an external system from inside a job produces externally visible side effects that cannot be hidden in this way. Thus, you have to worry about the results from partially completed jobs being visible to other systems, and the complexities of Hadoop task attempts and speculative execution.
    
8.  A better solution is writing the output to a file in HDFS using the job and then bulk importing that file into your DB.
    
9.  Also, in mapreduce since a job just has a map and reduce. You need to have multiple jobs with output of 1 as input to next and so, each job should be writing an output to a file which is just an intermediate state that will be read and then its of No use. The process of writing out this intermediate state to files is called materialization and its a big downsize due to memory constraints. (Thats why spark etc are better at this as they solve such problems as It usually writes the intermediate state between operations(map,filter,count etc) to be kept in memory or written to local disk, which requires less I/O than writing it to HDFS (where it must be replicated to several machines and written to disk on each replica). Also, In unix commands, we use pipes | which are basically stream the ouput of left command to right. But in mapreduce, we wait for whole 1 map reduce to complete to send it to next job. Even this problem, is solved in spark/Flink- as these are more like Unix-stream type.
    

The benefit of HDFS is its a distributed Filesystem and not a distributed db where you need to take care of data models, schema etc . File system means just byte sequences. To put it bluntly, Hadoop opened up the possibility of indiscriminately dumping data into HDFS, and only later figuring out how to process it further . By contrast, other databases typically require careful up-front modelling of the data and query patterns before importing the data into the database’s proprietary storage format.

Hive is basically, an SQL engine over HDFS using MapReduce Jobs.

* * *

## ==**STREAM PROCESSING**==

- The normal queues/message brokers are databases where producers produce in an order and consumers consume. But once a msg is consumed and ack is received by broker. The msg is deleted from the broker. So, unlike traditional DBs, data isnt stored for long time.
    
- Log Based Brokers -  Example - Kafka.
    
    - In this the brokers maintain a append only log record(log here means the messages queue only). And for scale, its also partitioned. And each partition maintains an 'offset' which is a monotonically increasing number telling the msg count. Each consumers tries to read messages from this offset.
    - ![Screenshot 2024-05-23 at 12.37.31 AM.png](../_resources/Screenshot%202024-05-23%20at%2012.37.31%20AM.png)
    - So the ordering in a partition is guaranteed but not across partitions.
    - If a consumer node fails, another node in the consumer group is assigned the failed consumer’s partitions, and it starts consuming messages at the last recorded offset. If the consumer had processed subsequent messages but not yet recorded their offset, those messages will be processed a second time upon restart. We will discuss ways of dealing with this issue later in the chapter.
    - Note that these logs/msgs are stored in disk and not in memory.
- ***CDC -***
    
    - There has been growing interest in change data capture (CDC), which is the process of observing all data changes written to a database and extracting them in a form in which they can be replicated to other systems(/DBs). CDC is especially interest‐ ing if changes are made available as a stream, immediately as they are written.
        
    - Change data capture is a mechanism for ensuring that all changes made to the system of record are also reflected in the derived data systems so that the derived systems have an accurate copy of the data.
        
    - Kafka Connect \[41\] is an effort to integrate change data capture tools for a wide range of database systems with Kafka. Once the stream of change events is in Kafka, it can be used to update derived data systems such as search indexes, and also feed into stream processing systems as discussed later in this chapter.
        
- EVENT SOURCING -
    
    - Designing Data-Intensive Applications
        
        Similarly to change data capture, event sourcing involves storing all changes to the application state as a log of change events. In change data capture, the application uses the database in a mutable way, updating and deleting records at will. The log of changes is extracted from the database at a low level. So this log just has for each primary key, the latest value. (Log compaction is the process that does this, removing older values and saving space).
        
    - While in event Sourcing, the application logic is explicitly built on the basis of immutable events that are written to an event log. In this case, the event store is append- only, and updates or deletes are discouraged or prohibited. So all the actions done are stored in same way/order.
        

* * *

## ==**FUTURE OF DATASYSTEMS =**==

- The traditional approach to synchronizing writes requires distributed transactions across heterogeneous storage systems \[18\], which I think is the wrong solution. Transactions within a single storage or stream processing system are feasible, but when data crosses the boundary between different technologies, I believe that an asynchronous event log with idempotent writes is a much more robust and practical approach.
- **Exactly-once** in databases operations means arranging the computation such that the final effect is the same as if no faults had occurred, even if the operation actually was retried due to some fault. We previously discussed a few approaches for achieving this goal.(As we dont want dual writes, like incrementing counter twice just becoz the write wasnt ack the first time although it got written in db)
    
    One of the most effective approaches is to make the operation idempotent . that is, to ensure that it has the same effect, no matter whether it is executed once or multiple times. However, taking an operation that is not naturally idempotent and making it idempotent requires some effort and care: you may need to maintain some additional metadata (such as the set of operation IDs that have updated a value), and ensure fencing when failing over from one node to another 
    
- PREVENTING DOUBLE WRITES - Even though you use transactions, it can happen that user clicked twice/retried twice. For the application, these are 2 separate requests, but we dont want user's money to be deducted twice for this we can use UUIDs
- For example, you could generate a unique identifier for an operation (such as a UUID) and include it as a hidden form field in the client application, or calculate a hash of all the relevant form fields to derive the operation ID \[3\]. If the web browser submits the POST request twice, the two requests will have the same operation ID. You can then pass that operation ID all the way through to the database and check that you only ever execute one operation with a given ID.
    

UNIQUENESS CONSTRAINT -

- say the usecase is uniqueness of a user name. We can solve it using stream based approach also using log based partition queues(kafka)
- A stream processor consumes all the messages in a log partition sequentially on a single thread(per partition) . Thus, if the log is partitioned based on the value that needs to be unique, a stream processor can unambiguously and deterministically decide which one of several conflicting operations came first. For example, in the case of several users trying to claim the same username \[57\]:
    
    1.  Every request for a username is encoded as a message, and appended to a partition determined by the hash of the username.
        
    2.  A stream processor sequentially reads the requests in the log, using a local database to keep track of which usernames are taken. For every request for a user‐ name that is available, it records the name as taken and emits a success message to an output stream. For every request for a username that is already taken, it emits a rejection message to an output stream.
        
    3.  The client that requested the username watches the output stream and waits for a success or rejection message corresponding to its request.
        
    
    It scales easily to a large request throughput by increasing the number of partitions, as each partition can be processed independently. The approach works not only for uniqueness constraints, but also for many other kinds of constraints. Its fundamental principle is that any writes that may conflict are routed to the same partition and processed sequentially.
    

Similarly, for a bank trasnfer which generally may need an atomic commit across records i.e. from payer to payee -  we can get equivalent correctness can be achieved with partitioned logs, and without an atomic commit:

1.  The request to transfer money from account A to account B is given a unique request ID by the client, and appended to a log partition based on the request ID.
    
2.  A stream processor reads the log of requests. For each request message it emits two messages to output streams: a debit instruction to the payer account A (partitioned by A), and a credit instruction to the payee account B (partitioned by B). The original request ID is included in those emitted messages.
    
3.  Further processors consume the streams of credit and debit instructions, deduplicate by request ID, and apply the changes to the account balances.
    

Since they are log partition queues, even if consumer goes down after debiting A only. when it comes up, it starts off from where left and the credit to A will happen.

* * *

&nbsp;
