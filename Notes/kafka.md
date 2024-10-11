https://blog.bytebytego.com/p/the-trillion-message-kafka-setup?utm_source=post-email-title&publication_id=817132&post_id=149639785&utm_campaign=email-post-title&isFreemail=true&r=1zlsga&triedRedirect=true&utm_medium=email

1.  Each partition is allocated to each broker. For scalability, there should be multiple partitions assigned to multiple brokers (all per topic). A thumb rule is 10Gb per partition. A broker will definitely will have multiple partitions, just that it wont have the same partition's leader and replica
    
2.  [drive](https://drive.google.com/drive/folders/1DWZA1b3nEwz7pSW0WrBL24SAOl8fO93U)
    
3.  There will always be a leader partition, and the rest are replica partitions. I/O will be done via the leader. DDL(CREATE , UPDATE AND DELETE TOPIC) will only by done by active controllers. While DML requests can be handled by all brokers.
    
4.  *Zookeeper* requires a quorum of servers to be up at any time. It uses a *majority* quorum to make any decision
    
5.  A broker is a computer/instance/container in a pod that runs the Kafka process. They can be thought of as machines that manage partitions, I/O, and have a disk that stores the data. Kafka broker is a server that runs Kafka and stores data logs. Kafka brokers store data on local disk. Kafka brokers split each partition into segments. Each segment is stored in a single data file on the disk attached to the broker.
    
    1.  Kafka cluster: A group of one or more Kafka brokers
        
    2.  Topics: Data is stored in topics, which are divided into partitions
        
    3.  Brokers: Brokers can hold multiple topics with different partitions
        
    4.  Segments: Each partition is split into segments
        
    5.  Data files: Each segment is stored in a single data file on the broker's disk
        
6.  It is the Zookeeper that performs the **election of the Kafka controller**. The controller is the Kafka broker responsible for defining the broker **leader** of each partition and the **followers**. For now, keep in mind that the broker leader is responsible for handling the requests for writing and reading a topic, and the broker followers are where the replication of events written in that partition takes place.
    
7.  Rack awareness? If the cluster is configured to be Rack Aware, then the leader partition is in one rack of brokers, and **almost** all the other replicas will be in another rack of brokers(Note that not always will this be possible/done thats why **almost**). If one leader partition goes down and another replica (ISR) becomes the leader now, the replicas in that rack of this new leader are not automatically transferred to the other rack. Rack awareness is compromised in this case. If you don't want this, have multiple brokers so that each partition is in each rack. Each **broker** and **consumer** is assigned to a **rack**, which represents a logical grouping of machines that are physically located close to each other.
    
8.  In Sync replica (ISR)
    
9.  The active controller (one of the Kafka brokers) keeps track of the health of all leader partitions via Zookeeper. If one of them fails, the controller checks if there is any ISR replica for this partition. If yes, it will promote it to leader and make the leader a replica, updating this data to Zookeeper. Zookeeper will notify all brokers, and they will refresh their metadata about partitions. If there is no ISR, then producers/consumers can't produce/consume since there is no leader. An admin should manually make any out-of-sync replica a leader. But in this case, there will be some data loss (the leader and the out-of-sync replica may be 1 hour apart in data).
    
10. All brokers send a heartbeat to the Zookeeper zNode, and one of the brokers, the active controller, keeps reading it. That's how the active controller knows about other partitions.
    
11. When writing to Kafka via a producer in some app, you create a producer object by specifying some properties. One such property is bootstrap-servers, which is the list of IPs of brokers. The write request is sent to one of those broker IPs. Note that no load balancing or broker health check is done by the Java app itself. It just sends equal requests to each broker without checking latency, health, traffic, etc. If you want these explicitly, have an ELB before brokers and use that in your app. Once any broker receives a request, based on which partition the input message will be assigned (decided based on below strategy), the request will be redirected by this broker to the owner of that partition (the leader partition must be assigned to one of the brokers, so it is redirected to that broker). Once this leader write is done, then the requests to brokers to write the replicas will be sent. And that replica broker will itself read from the leader and write it to itself.
    
12. For a topic, there's a property called min.insync.replicas (generally 2 with replicationFactor=3). This means that out of 3 copies, there's one leader, one ISR, and one out-of-sync replica (OSR). While writing, the acknowledgment of the message will be given if it is written to 2 replicas (if the ack property is set as 'All', it can be 0/1/'All'). 0 means instant, 1 means written to the leader and ack given, and All means written to the leader and ISR's and ack is given.
    
13. Note that a replica is not hard-coded as ISR/OSR. Once the leader write is done, the writes to the other 2 replicas start in parallel. Whichever replica write is done first, it becomes the ISR. If after some time, the other replica write is also done, then it also becomes an ISR. We always want all replicas to be ISR, but due to some issues/lags, a replica can become OSR and with time it can be many messages behind of the leader. And may be after sometime, after all msgs written - it will again become ISR
    
14. Kafka Default partitioning strategy - Either uses the message's (Key-Val) key hash or, if the key is null, then the round-robin strategy.
    
15. In Kafka, order can only be guaranteed within a partition. This means that \*\*if messages were sent from the producer in a specific order, the broker will write them to different partitions and when a consumer reads, it reads at a time from 1 partition only, and orderly in that partition. \*\*So, across partitions consumer cant read in order as its written.
    
16. Kafka brokers write messages to memory(/flush) and after 1sec/10k msgs which ever occurs first, they are written to file system.
    
17. Deadlater topic - All the messages that have some problems(like schema validation fail) in it arent directly written to main topic and are written to the deadlater topic. Consumers can read from this topic and can take calls and debug why it failed.
    
18. A producer config. - enable.idempotence = true to avoid duplicate retries to happen (takes more latency but retry duplication is avoided)
    

Producer async call back -

1.  producer.send(record, new CustomCallBack());
    
2.  The main thread of your application calls the `send()` method to send a message. This method does not wait for the acknowledgment but returns immediately.
    
3.  Background Thread Handles Communication: Internally, the Kafka producer has background threads that handle communication with Kafka brokers. These threads are responsible for sending the message to the broker, handling acknowledgments, and managing other aspects of communication. So this background thread will be waiting for kafka to respond.
    
4.  Acknowledgment Handling: When the Kafka broker receives the message and successfully processes it, an acknowledgment is sent back to the producer. The background thread in the producer receives this acknowledgment.
    
5.  Callback Execution (Optional): If you've specified a callback function when calling `send()`, the callback is executed by the background thread once the acknowledgment is received. This allows your application to take specific actions based on the success or failure of the message delivery.
    

Nosql dbs are highly scalable and highly avaialble and if we need key-val lookup, then redis, cassandra, mongodb are used.

- Mirror Maker - A kafka utility helpful is replicating topics from 1 cluster to another. You specify the consumer and producer .prop files and run the mirrormaker command with those props, and replication will start.
- Kafka Connect - A kafka utility to transfer data from rdms/hdfs/ etc files system to and from kafka

Kafka Stream -

1.  Say onincoming msgs of customer data , u need to find the running count of customer's how many products purchased. State is to be managed since, we keep count of previous count
    
2.  So, instead of putting in external db, Kafka stream api provided their own db called RocksDB which is like redis and maintains the key-val
