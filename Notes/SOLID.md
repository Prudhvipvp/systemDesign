

S - Single Responsibility Principle -

- Every class/method must be responsible for one type of behaviour i.e. a single method / class must not be used for different behaviours (using say if else condition).

O - Open Close Principle -

- Your code should be Open for extension and close for modification i.e. whenever u extend the code, u shouldn’t be making a modification in already been written parent class

L - Liskov Substitution Principle -

- The object of the parent class must be completely replaceable by its child class (i.e. say fly() method of bird class isn't used/not supposed  in its child ostrich class). So, fly() shouldn't be part of bird class and should be part of a new interface and every child bird which can fly will implement the interface

I - Interface Segregation Principle -

- An interface need not have all the methods and have thin/lean and don't put all the behaviours in a single interface and have separate interfaces for it. Anyways a class can multi implement interfaces.

D - Dependency Inversion -

- Within a class A, if we use/create a new class B, then A is dependent on B. Now if later we need to use a class C instead of B, then lot changes should be done. So, better to create a base class/interface and have the class B or C in A. Now just the interface reference and use the constructor of A to create new class like -
- So, in below way we are not creating new class in here and any new class if made that should be passed to the A's constructor. So we are inverting the dependency using the constructor / a method/ etc.

```
class A {
    BaseclassInterface i;
    public A(BaseclassInterface i){
        this.i=i;
    }
    / / /
}
```

- The dependency inversion principle - So, finally, Instead of creating a concrete class (B/C) in an other class A, create the abstract class (interface here) to remove the dependency. So, basically a concrete class (A here) should depend on an abstract class only and not other concrete class.
- DEPENDENCY INJECTION( we did using constructor above) is a way to achieve dependency Inversion.
- Somewhere a new object should be made, so it can be made using a factory class.
- Say, 1000s of classes and dependent on each other, so a dependency graph will be there, now to create an object, we need to create all the objects in order by seeing the dependency graph ( the order is topological sort). Now, by using frameworks like MVC, Spring ioc , they do this for us by analysing graph and creating objects etc.. So, this is INVERSION OF CONTROL (from a developer to framework).

## ***CAP:***

1.  **Consistency**: All nodes in the distributed system see the same data at the same time. In a consistent system, there is a guarantee that once a write is acknowledged, all subsequent reads will reflect that write.
    
2.  **Availability**: Every request to the distributed system receives a response, without a guarantee that it contains the most recent version of the information. An available system ensures that there is a response to every request, even if it might not be the most up-to-date data.
    
3.  **Partition Tolerance:** The system continues to operate and provide some level of service despite network partitions (communication failures) that might occur between nodes in the distributed system. Partition tolerance is about the system's ability to handle network failures and still function. Network partition means if u have 10 nodes and a network partition occurs, then nodes 1 to 5 form a partition and nodes 6 to 10 form partition 2 . The 2 partitions cant communicate with each other.
    

- Mysql has C and A.
    
- Hbase has A and P. P is becoz, HBase replicates data across multiple region servers to ensure fault tolerance. Each piece of data is typically replicated to multiple nodes in the cluster. If a region server becomes unavailable, the system can continue serving requests from the replicas.  Also, the underlying hdfs is designed to have replication and distributed. The zk also keeps check of all region servers, if any of them dies, it elects a new region server.
    
-   
    **ACID** - (?)
    
- **Atomicity**: A transaction is an atomic unit. All of the instructions within a transaction will successfully execute, or none of them will execute.
    
- **Consistency**: A database is initially in a consistent state, and it should remain consistent after every transaction.
    
- **Isolation**: If multiple transactions are running concurrently, they shouldn’t be affected by each other, meaning that the result should be the same as the result obtained if the transactions were running sequentially.
    
- **Durability**: Changes that have been committed to the database should remain, even in the event of a software or system failure. and **BASE**
    
- RDBMS(Like mysql) follow ACID and Nosql DBs follow BASE properties
    
- **DB Sharding** -  Sharding is just horizontal scaling of a db i.e. distributing your one machine's data into multiple smaller machines to improve scale and limitless horizontal scaling.
    
- DB partitioning - Partitioning is dividing a table into multiple portions i.e. a table with 5 columns in mysql can be broken into two tables with 4 columns  and 2 columns. The 2 columns one will be stored in memory for better latency issues and 4 columns one will have a common key with the other table.
    
- **In a distributed system**, a leader node is required to take a single decision and maintain sync among nodes. With a leader, it is easier to maintain a single source of truth, reducing the risk of conflicting updates or decisions.
    
- Column oriented databases, under the hood, store all values from each column together whereas row oriented databases store all the values in a row together(A good way to visualize this is to think about it as the values of each column are stored in separate files.)
    
- Cassandra uses a wide column-oriented database model, which means that rows do not have to have the same columns. A row of data can have any number of columns, each with a name and a value.