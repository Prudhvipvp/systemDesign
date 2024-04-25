HLD Probs-

For the designs where we use object storages, its better that the service returns just the metadata to the client, like the image/video path only. And the client then loads that image via CDNs via the object storage

When uploading a image to server/message to server. Basically, treat it as a txn. For each upload, a unique id has to be generated and once the server receives the image , it has to return a jobId back to client. Now server also need to cache the txnId and this job id for some time. So tht duplicate requests are not done.

Also one of http protocol is Chunked transfer encoding mechanism because it allows you to resume an upload (after a pause or an internet disconnection).I<span style="color: #ffffff;">n chunked transfer encoding, the data stream is divided into a series of non-overlapping "chunks". The chunks are sent out and received independently of one another.</span>

<span style="color: #ffffff;">Object storage are meant for media files instead of storing in say rdbms, as say doesnt fit in a simple varChar datatype. They are distributed, highly horizontal scalable, quick r/w and also handle replication easily.</span>

<span style="color: #ffffff;">For watching video, today in youtube also. Chunks of video is sent to client and as video progresses, more chunks are fetched and loaded in memory of client. Note that audio and video are separately fetched.</span>

<span style="color: #ffffff;">Nosql is more scalable than sql. Advantage of sql is structured and joinable data and transaction management(<span style="color: #e7f4e8;">set of database operations that are executed as a single unit, known as an ACID transaction - A -</span></span> Ensures that either all operations within a transaction are executed completely, or none of them are), <span style="color: #ffffff;">consistency.</span>

<span style="color: #ffffff;">In cases where each user is connected to server (long polling/websockets) - you have to maintain a session service which tells what box a user is connected to for efficient msg delivery. Similarly for db that are shared, its better to have something like a directory service that tells where a key is</span>

<span style="color: #ffffff;">A gateway service like mapi just does api routing to your internal svc, authentication, rate limiting, response aggregation. Basically these are the ones which keep a long poll connection with the clients(In messenger kind of case). So its better that gateway just remains as a stateless and a dumb service. Most logic should be rather on internal services. As gateways are already expensive as they maintain a connection and already has a good memory being used. Basically gateway svc is called reverse proxy.</span>

```
Long polling and WebSockets are both push-based methods for creating live experiences for users online. Both are supported on all web browsers. However, they differ in the following ways:
Connection type: 
Long polling uses a series of intermittent HTTP requests, while WebSockets establish a persistent, full-duplex connection.
Latency: WebSockets generally offer lower latency compared to long polling.
Real-time capability: Long polling offers near real-time communication but with inherent latency due to the request-response cycle. 
WebSocket enables true real-time communication with minimal latency.
Resource intensity: Long polling is more resource intensive on the server than WebSockets.
Latency problems: WebSockets eliminates latency problems that arise with long polling.
Data transfer: WebSockets allow for more efficient data transfer.
Message ordering: Reliable message ordering can be an issue with long polling.
```

Basically, long polling waits and keeps connection alive with server till a new msg is there/timeout. And once new msg has come, the connection closes and new one has to be made.

While web sockets maintain a constant bi directional connection which is realtime and low latency. Low communication bcoz we do handshake only once (Unlike long polling where every time u make a new connection when a new msg is delivered(/timed out)).

There is a 3rd one called - SSE(Server Sent Events) In this client establishes a persistent and long term connection with server and its only for server to client(uni direction) and server can keep on sending events to clients once the conneciton is made. If the client needs to send data, then it has to make a new connection using some other protocal may be simple HTTP.

Whenever we maintain long polling with millions of users, the ELB/gateway via sessionSVC should direct a request to a user to correct server thats holding the poll request.

Cache policies -

1\. Write through - latent as writes to both cache and disk. But consistent as written both places. Used when write volume is less. Latent but very secure, no data loss

2\. Write back - written to cache only. and at a later time (when that cache row is going to be evicted from cache). Faster but risk of data loss in case cache server fails.

3\. Write around - written to db only and no cache . This helps in preventing cache being flooded with writes. But if recently written data is read, then cache miss happens and slower read from db.

In CAP, You should almost support P always as network partition shouldn't stop your service. So the call comes down to consistency vs availability. <span style="color: #e6edf3;">CP is a good choice if your business needs require atomic reads and writes. <span style="color: #e6edf3;">AP is a good choice if the business needs to allow for</span> <ins>eventual consistency</ins> <span style="color: #e6edf3;">or when the system needs to continue working despite external errors.</span></span>

&nbsp;

Read replica - there s a master that handles writes and writes are done async to read replicas also. The reads are done via read replica.

Consistent hashing is done for better data distribution for sharding and less data transfer.

We denormalise data in sql which means storing same column in different tables , so more data but benefit is no need of joins. Join during sharding is expensive as different servers. Once data becomes distributed with techniques such as <ins>federation</ins> and <ins>sharding</ins>, managing joins across data centers further increases complexity. Denormalization might circumvent the need for such complex joins. In most systems, reads can heavily outnumber writes 100:1 or even 1000:1. A read resulting in a complex database join can be very expensive, spending a significant amount of time on disk operations.

In NOSQL, <span style="color: #e6edf3;">Data is denormalized, and joins are generally done in the application code. Most NoSQL stores lack true ACID transactions and favour</span> <ins>eventual consistency</ins><span style="color: #e6edf3;">. NoSQL follows BASE means it <span style="color: #e6edf3;">chooses availability over consistency. One such - <span style="color: #e6edf3;">Key-value stores provide high performance(O(1) R/W)  and are often used for simple data models or for rapidly-changing data, such as an in-memory cache layer. Since they offer only a limited set of operations, complexity is shifted to the application layer if additional operations are needed. MongoDB is document store and not key-val. Document stores provide high flexibility and are often used for working with occasionally changing data. Aerospike is key-val DB</span></span></span>

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;">In nosql , sharding comes in build. While in sql, you have to do it and manage implement node failures via master-slave etc.</span></span></span>

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;">So if you need data consistency via acid and if u dont see much data growth for future , go for sql dbs</span></span></span>

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;">If data is increasing and with no proper schema - NOSQL.</span></span></span>

A relationaldatabase works well with read-heavy and write less frequently workflow.  
This is because the number of users who visit the hotel website/apps is a few orders  
of magnitude higher than those who actually make reservations. NoSQL databases  
are generally optimized for writes and the relational database works well enough for read-heavy workflow.

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;">Wide column DBs like Hbase, cassandra <span style="color: #e6edf3;">maintain keys in lexicographic order, allowing efficient retrieval of selective key ranges and</span> offer high availability and high scalability. They are often used for very large data sets.</span></span></span>

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;">Cache Invalidation - means if the data is changed in db, the data now present in cache is invalid and has to be invalidated. So write thru, write back and write around are solutions for thisl</span></span></span>

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;">BACK PRESSURE IN queues -</span></span></span>

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: rgba(255, 255, 255, 0.9);">Back pressure works by providing feedback from consumers to producers about their processing capacity. When a consumer can't keep up with the incoming message rate, it signals the producer to slow down, preventing message overload and potential data loss.</span></span></span></span>

&nbsp;

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: rgba(255, 255, 255, 0.9);">In sql tables, to faster read build index over a column over which generally the read condition is made.</span></span></span></span>

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: rgba(255, 255, 255, 0.9);">Dont use lists as columns in sql as much as possible.</span></span></span></span>

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: rgba(255, 255, 255, 0.9);">To avoid elb single point of failure. We can do it using dns and directing requests for our domain to diff elb's. Or we can have a active-passive elb cluster where passive takes over if active fails</span></span></span></span>

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: rgba(255, 255, 255, 0.9);">Proxy vs Reverse Proxy -</span></span></span></span>

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: rgba(255, 255, 255, 0.9);">Proxy is after the clients, it hides the user info and hits the internet as if request came from proxy server.</span></span></span></span>

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: rgba(255, 255, 255, 0.9);">Reverse proxy is a proxy to our internal services. So client requests and it comes over internet to our dns that hits first our reverse proxy and then this hits our internal services.</span></span></span></span>

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: rgba(255, 255, 255, 0.9);">There will always be a gateway Service - generally shown as a single. Now gateway does auth and then calls other elb which calls backend server. In cases where clientB request needs to be routed to serverB, In this case ELB wont be involved, gateway will figure it out and directly forward that request to serverB.</span></span></span></span>

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: rgba(255, 255, 255, 0.9);">And when persistent connections(http poll/websockets) are maintained, they are made by client directly with backend servers via gateway ( so no elb in between, the gateway itself is kind of elb routing appropriate clients to servers).</span></span></span></span>

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: rgba(255, 255, 255, 0.9);">![Screenshot 2024-04-08 at 10.29.13 PM.png](../_resources/Screenshot%202024-04-08%20at%2010.29.13%20PM.png)</span></span></span></span>

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: rgba(255, 255, 255, 0.9);">The API Gateway can also be in practical - multiple instances (like mapi) for fault toleration and scalability . Now before gateway there can be ELB which routes traffic/ DNS Resolution to multiple public VIPs.</span></span></span></span>

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: rgba(255, 255, 255, 0.9);">Why reverse Proxy -</span></span></span></span>

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: rgba(255, 255, 255, 0.9);">1\. Different protocols for client and for your internal services, you may not need all big headers.</span></span></span></span>

<span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: #e6edf3;"><span style="color: rgba(255, 255, 255, 0.9);">2\. Auth, Caching and</span></span></span></span> Routing: centralize traffic to internal services and provides unified interfaces to the public. For example, [www.example.com/index](http://www.example.com/index) and [www.example.com/sports](http://www.example.com/sports) appear to come from the same domain, but those pages are from different servers behind the reverse proxy.

<span style="color: #ffffff;"><span><span><span><span>Consistent hashing is a very useful strategy for distributed caching system and DHTs. It allows us to distribute data across a cluster in such a way that will minimize reorganization when nodes are added or removed. Hence, the caching system will be easier to scale up or scale down. So when a new cahce server is added, using consisten hashing only K/N of keys have to be re distributed (K is total keys, n is total servers)</span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span style="color: #e7f4ee;">The master and slave systems can also be kept in sync by using binary logs or transaction logs. <span style="color: #ffffff;">Asynchronous replication allows the master to continue its operation without waiting for acknowledgment from the slave. While this enhances performance, it may slightly delay data consistency. Whereas, synchronous replication ensures that the master waits for acknowledgment from at least one slave before considering a transaction as committed. This provides a higher level of data consistency but may impact performance due to increased latency.</span></span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span>Things to consider =</span></span></span></span></span>

```
PRODUCT REQUIREMENTS AND NFR NUMS - ASK LATENCY, sla, up time AND THRU PUT
DATA MODEL - SQL ,NO SQL
API DESIGN
HIGH LEVEL ARCHITECTURE
BOTTLE NECKS FOR SCALABILITY(HOT SHARDS), CACHING AND CACHE INVALIDATION, indexing , ELB, AUTO SCALING, traffic PEAKS
singlw point of failure, 
DB Cleanup if applicable
FAULT TOLERANCE (for elb also)
```

&nbsp;

<span style="color: #ffffff;"><span><span><span><span><span>At a high-</span><span>level, our web servers will manage users’ sessions and application servers</span> <span>will handle all the ticket management, storing data in the databases as well as working with the cache servers to process reservations.</span></span></span></span></span></span>

Consistent hashing also tried to distriubute almost uniform, so in cases where sharding is done on userId and if single user has many messages, in such cases CH will try to distribute among nodes uniformly. So that any new node will need least re distribution. Uses HASH ring.

&nbsp;

Websockets -

![Screenshot 2024-04-10 at 11.40.54 PM.png](../_resources/Screenshot%202024-04-10%20at%2011.40.54%20PM.png)

So, user A sends to LB and to chat Service it comes, msg is stored and acked and from sessions we get chat ServerId where userB websocket is connected to and sessions can re route that request to that server and it will send msg to userB.

1.  ## <span style="color: #ffffff;"><span><span><span><span><span>TicketMaster -</span></span></span></span></span></span>
    

<span style="color: #ffffff;"><span><span><span><span><span>A city will have multiple Cinemas and each cinema will have multiple halls and each hall will have multiple Shows. Each show is mapped to a movie.</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>Shard based on showId to prevent movieId HOT sharding</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>1\. To store the active reservations that are ONHOLD state we have a activeReservationService which holds in memory in a linkedHashMap datastructure &lt;ShowId, <BookingId, timestamp&gt;> these are ordered in timestamp. So if the head is more than 5 mins, its removed from map. Also, if any booking id is confirmed its quickly removed from map. Obviously , its also stored in the DB. where each bookingid has a status telling Reserved/Booked</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>2\. Simultaneously, we also have a waiting User Svc which stores in a linkedHashMap, the waiting users in FIFO order and LongPolling is maintained from client to server to notify whenever seat is available.</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>Also booking concurrency is achieved via transactions(Serializable isolation level - highest), while booking a write lock is acquired so that no other can write.</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>Now how is the activeReservations and waiting User service servers work? By consistent hashing to allocate the appropriate app servers for both these servers so that a particular showID's requests always go to a set of app servers. Now whenever in one of the server, booking gets booked/Cancelled. It has to be notified to all the waitingUserSvc servers/the servers where showId is allocated to. The webservers handling user sessions. The ELB has to do the consistentHashing for this above allocation.</span></span></span></span></span></span>

* * *

## <span style="color: #ffffff;"><span><span><span><span><span>2.Proximity svc -</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>Given a lat long, give me near by restaurants etc.</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>Simply doing sql query where ur table contains lat long of all restaurants is not optimal.</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>If convert whole maps to square grids of size d kms. Then given a lat long, we can get its gridId and just search in that grid and 8 surrounding grids. But due to fixed grid size, the sparse and dense places will be effected.</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>So better solution is using Dynamic sized grids called 'Quad Trees'. So the root node is the whole world, and we keep breaking each node into 4 child nodes till each of the node con</span></span></span></span></span></span><span style="color: #ffffff;"><span><span><span><span><span>tains no more than 500 places in them. This way all child nodes of tree will contains the places. While searching its simple O(height) search and to get neighbour grids, we need to maintain a doubly linkedlist of all the child nodes.</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>now the logic is not 8 neighbour grids, rather -> if current grid where given lat long lies contains required number of search results return it. Or else keep looking in neighbours till the limit is reached/ the max radius is touched.</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>Size -> say we have 500M locations. And in each leaf node we store locationId, lat, long -> each 8Bytes. So net - 12GB.</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>Now each grid to have 500 places -> so total 1M leaf grids. Now estimate is if we have 1M leaf nodes. Almost 1/3rd of them will be present in the rest of tree. and rest tree nodes will just have 4 pointers(each 8 bytes). So</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>1/3\*1M\*4\*8 = 10MB. So total quad tree would fit in Memory(12.01Gb).</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>So along with storing in db (sql/mongo) we will have this tree also for faster search.</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>But still if read traffic increases and quad tree size increases, we need to partition. Simple sharding may cause hotness. So partition based on locationId and each server will have a different quad tree. When a query is made its made to all servers and a aggregation services now should find the best grids among these search results.</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>Last one is fault tolerance -> we will have replica of quad tree server, so the slave will do reads and master does all writes.</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>What if both dies? We can re build the quad tree in the server again by querying our db. but we need to iterate thru all locationIds generate hashes and assign to our serverId.</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>Instead we can have a other store that stores reverse index. I.e. a key val , where key is quad tree serverId and val is list of locationIds on it. This way we can quickly re build the tree.</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>To add ranking to our results, We can store rating of place in both db and quad tree and include this rating also to get top results and aggregation server can also help here to filter top rated.</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>Caching-> the search results are primarily given by server directly from memory, so not much of a latency there, but we can have a cache before the db as the db is the one which gives us meta details from a locationId.</span></span></span></span></span></span>

* * *

## <span style="color: #ffffff;"><span><span><span><span><span>3\. Uber -</span></span></span></span></span></span>

<span style="color: #ffffff;"><span><span><span><span><span>Its same as above, but the quadtree will keep changing as the driver will keep moving. So more write heavy also. For this</span></span></span></span></span></span>

1.  locationId is driverId here
    
2.  Every driver will be connected to the server and send updates about his latest location every 3 seconds.
    
3.  Now driver will send these to a separate server say driverServer, and these server will have a hashTable -> driverId - timestamp - latest lat long. (Here also mulitple servers and same driverId based sharding can be used).
    
4.  Now this driverServer will notify the quadTree server about the driverUpdates that happened in last 10 seconds and for all those driverIds -> the quadTree servers for those will get updated i.e. the tree will be updated. So this way around every 15 seconds the quadTree will keep updating.
    
5.  Along with updating driverLocation, whenever a user requests for a ride, he sees the live location of all the drivers near By. This can be done by a notificationService.
    
6.  Whenever user opens app, the notificationSvc will add his userId to the subscribers list to the drivers in that Area. So using this subscriber consumer model, the notifSvc will read from the driverSvc the latest driver location Updates and send them to the subscribed users only. And similarly location of users to drivers.
    
7.  Obviously, for driverSvc also as we are storing hashTable we will have a persistent storage like an SSD behind it for fault toleration.
    
    ![Screenshot 2024-04-07 at 3.15.04 PM.png](../_resources/Screenshot%202024-04-07%20at%203.15.04%20PM.png)
    
    * * *
    

&nbsp;

## 4\. News Feed Design -

1.  You will have users, pages, groups, posts, stories.
2.  So multiple tables and better to have NOSQL for availability and scalability.
3.  Simply put when u need the feed -> you go to all users followers and get latest posts and their meta and rank them using ranking service and return them. But doing this is very slow.
4.  So we would go with offline Feed Generation, where in the feed is generated periodically for all users(or in a smart way a set of users and LRU cache is used / based on time of day who is active only their feed is generated).
5.  This feed will be kept in our cache servers sharded by userId to Map&lt;feedId, feed&gt;, lastUpdatedTime. So periodically, we will keep fetching feed from the latestUpdated time to current time and update our cache.
6.  Assuming we have 300M DAU and for each user lets say we store 500 posts in cache . Each post say 1KB. total 150TB of memory. If a cache server holds 100GB we need 1500 machines.
7.  Since cache holds 200 posts per user, whenever user asks we give top 10 and the cache's hashMap anyways stores in order of relevancy and whenever user scrolls, he will give the last post and next 20 posts from that will be given.
8.  Obviously s3 is used for objects and No need of cdn as already cache m its stored.
9.  Now whenever a user posts, request comes to feedGenSvc and it stores it in DB and also update the cache. can have queue also
10. Theres a Feed notification Service now which will notify this change in feed for all the followers. Now this can be pull model, all users will pull the new feed if present but most times it may be empty. Push is better but for celebrities its difficult.So hybrid of push + pull. Push for non celebs and pull for celebs can be used.![Screenshot 2024-04-07 at 3.52.39 PM.png](../_resources/Screenshot%202024-04-07%20at%203.52.39%20PM.png)

## 5.WEB Crawler -

![Screenshot 2024-04-07 at 5.02.59 PM.png](../_resources/Screenshot%202024-04-07%20at%205.02.59%20PM.png)

Basically, dns resolver to get ip from dns.

URLFrontier that has a queue fifo to which like a BFS Unseen links will be kept on adde

HTML Fetcher is a module to fetch html doc from the webpage. It can extensible to Video pages etc

Document Input stream is a cache where the documents are stored in its local cache for small files and in a db for big. Its basically used so that multiple modules can now read the docs without downloading again.

Document de dup - it basically, calculates checksum of current doc and if present in checkSum cache(/ checkSum db if cache miss) we remove this doc .

Now the extractor module just pulls doc from DIS and calls URL Filter which can filter some documents/url blacklists and then gives to url Dedup which basically again checks from url checksum if this doc is already seen.

and finally its added back to url frontier.

We might have to crawl 15Billion web pages in 4 weeks. and considiering checksum is 8bytes, its just 120GB for doc checksum cache.

## 6\. TWITTER SEARCH -

1.  The twitter has almost 400M new tweets per day with 800M DAU and each tweet around 300 bytes. There will be text searches of 500M per day containing AND/OR conditions also.
2.  ![Screenshot 2024-04-07 at 5.25.39 PM.png](../_resources/Screenshot%202024-04-07%20at%205.25.39%20PM.png)
3.  Our tweets and objects will be stored in db. But for faster search we need to build index on words. Word and all the tweetIds containing this word. And store this in memory.
4.  Say we have 500K words english and in memory we are storing past 2 years tweets -> so 292B tweets. Each tweetId 5Bytes.
5.  So 292B\*5 = 1460GB of tweetIds total. Now for 500K words(each word 5Bytes) also we need to store, 500K\*5 - 2.5MB
6.  Assuming each tweet will have around 15 significant words(without prepositions and conjunctions). So each tweetId will be stored 15 times. So 1460\*15 +2.5MB = 21TB
7.  Each server can store around 144GB Memory. So total 152 servers are needed.
8.  Now on what basis are these servers sharded? if its word , then hot servers will be there and very much non uniform distribution.
9.  So better of tweetId. Now, if we want to search, we search from all indexServers and they return to Aggregator server which gets top results and returns
10. In the same way as uber, for fault tolerance, if index Server dies to re build it , its optimal to have a separate hashtable stored in mem of index Builder server. It will have indexServerId to HashSet of tweetIds. Obviously replicas in both cases.
11. A cache before db which will give the meta data of a tweetId the content, username etc..

* * *

## 7\. Rate Limiter

Algos = fixed window, sliding window, TOKEN BUCKET ALGO =  In this bucket has 3 tokens and each request is given a token, tokens are refilled at a constant rate of R / second. Now how does distributed pods work?

Maintain the bucket in a redis cache/  Have buckets in all pods and let pods talk to each other using communication protocols can be a leader slave/ GOSSIP Communication via UDP, RAFT etc. Basically, the pod ASKS the message broadcaster to broadcast message to all pods to get their tokens in their buckets. So that we can take a call.

Basically, by knowing whats the count in each bucket pod, we reduce the tokens that gets used in other buckets in our bucket also.

&nbsp;

&nbsp;

## 8.YOUTUBE-

while uploading via queue -> its picked by encoder that encodes it into mulitple formats and thumbNailGenSvc that generates and stores and A video deDup service at the upload time which will prevent any existing video upload by some Video Matching algorithm intelligently.

Sharded based on videoID as if done on userId , it can become hot. But hot video can also  happen, so use cache also. CDNs

## 9.MESSENGER -

1.  The servers maintain a long poll request with all clients, there will be multiple servers and ELb will maintain hashtable containing userId to serverId map. A server can around maintain 50K open connections
2.  Whenever user sends msg to user B. ELB transfers it to corresponding server and server will first send it to the userB and then acks the userA and then store it in the DB.
3.  DB cant be sql or mongodb as we will have multiple small messages in short time. So better to use HBase, as it has a mechanism where it stores all messages in its buffer and then dump it to disk periodically. Also fetching msgs from row Key and prefix Scans can help.
4.  How to maintain order of msgs? sequence number is maintained by each client separately. Or else timestamp is sent by client(not server) so whoever pressed the sent button first their msg will be seen first.
5.  ![Screenshot 2024-04-07 at 10.19.42 PM.png](../_resources/Screenshot%202024-04-07%20at%2010.19.42%20PM.png)

## 10.DROPBOX -

1.  Basically, the client should keep track of all the changes happening in file. It breaks the file into small chunks and see incremental update in each chunks. And only these incremental chunks are transferred to server(cloud storage). And these are put in a queue (with same partition ID per file).
2.  Now consumer consumes it and hits the mapper svc which will give from in memory what all clients have to be notified about this file. and then hits with these clients to the notification svc elb, which behind has multiple servers maintaining long poll request, Now in this way each client receives updates.
3.  Client will have its components like Watcher that watches for updates from notif Svc / local changes and a Chunker that chunks file.

For use cases, where most recent photos etc are needed, have timestamp part of photoId and shard on photoId.

## 11\. TYPE AHEAD SUGGESTOR -

![Screenshot 2024-04-08 at 2.03.10 AM.png](../_resources/Screenshot%202024-04-08%20at%202.03.10%20AM.png)

Basically, trie will be formed with each child node being the word end and also has the frequency of how many times this word has been searched.

Each cache server will store a trie and it can be partitoned say into 4 servers , ALL  words from A-M go to server1 and so on

Now we also need to store the user searches to update our tries.This can be done by separate collectionService, it keeps collecting logs of what user entered and dumps them to object storage and periodically, aggregator picks them does some MapReduce stuff to get the hash table where word to frequency is calculated and stored in cassandra. Finally, trie builder will build the trie and update in the redis trie cache

&nbsp;

### HOW are objects uploaded from clients -

<img src="../_resources/Screenshot%202024-04-13%20at%209.15.53%20PM.png" alt="Screenshot 2024-04-13 at 9.15.53 PM.png" width="847" height="661" class="jop-noMdConv">

As seen, when the uploadVideo() api is hit, the meta data is hit to the elb and meta data stored in db. But the actual blob which is video, is directly uploaded to the blob storage, say orginal storage, from this the transcoding servers pull and do thumbNail generation/ Encoding etc and add them to transcodedStorage and CDNs.

&nbsp;

<span style="color: #ffffff;"><span>Document Input stream - A DIS is an input stream that caches the entire contents of the document read from the internet. It also provides methods to re-read the document. The DIS can cache</span> small documents (64 KB or less) entirely in memory, while larger documents can be temporarily written to a backing file.</span>

<span style="color: #ffffff;">Streams is kafka - which retains the data even after consumption. So replay capability is there.</span>

<span style="color: #ffffff;">Handle spiky traffic with a queue.</span>

&nbsp;For repetitive queries from clients, clients send a reservationId everytime unique to server, now when a user clicks the button repeatedly, client should understand and send same reservationId and the backend has a check that every time a unique reservationId only has to be sent.

For queue fault tolerance. Implement <span style="color: #ececec;">retry mechanisms with exponential backoff and jitter to handle transient failures. Have replica queues as a backup and queues should have persistent storage, so that data is not lost if queue dies.</span>

<span style="color: #ececec;">in kafka, the same partition exists across different servers/brokers. 1 will be leader other will be replicas.</span>

<span style="color: #ececec;">For consistency ACID across different tables/DB nodes. How? say you made a reservation entry in a table and then trying to add another entry in other table corresponding this reservationId and if it fails. The first commit should also be rolledBack. For this we have -</span>

1\. Two-phase commit (2PC) \[12\]. 2PC is a database protocol used to guarantee atomic transaction commit across multiple nodes, i.e., either all nodes succeeded or all nodes  
failed. Because 2PC is a blocking protocol, a single node failure blocks the progress until the node has recovered. It's not performant. Basically, the whole 2PC works as a single atomic transaction.  
• Saga. A Saga is a sequence of local transactions. Each transaction updates and publishes a message to trigger the next transaction step. If a step fails, the saga executes  
compensating transactions to undo the changes that were made by preceding trans- actions \[13\]. 2PC works as asingle commit to perform ACID transactions while Saga consists of multiple steps and relies on eventual consistency. So for sometime, the older tables will still see older data.

<span style="color: rgba(255, 255, 255, 0.9);">1\. Understand the functional and non-functional requirements before designing.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">2\. Clearly define the use cases and constraints of the system.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">3\. There is no perfect solution. It’s all about tradeoffs.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">4\. Design the system to be flexible.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">5\. Assume everything can and will fail. Make it fault tolerant.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">6\. Avoid over-engineering.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">7\. Design your system for scalability from the ground up.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">8\. Prefer horizontal scaling over vertical scaling for scalability.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">9\. Use Load Balancers to ensure high availability and distribute traffic.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">10\. Consider using SQL Databases for structured data and ACID transactions.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">11\. Opt for NoSQL Databases when dealing with unstructured data.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">12\. Consider using a graph database for highly connected data.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">13\. Use Database Sharding to scale SQL databases horizontally.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">14\. Use Database Indexing and search engines for efficient data retrievals.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">15\. Use Rate Limiting to prevent system overload and DOS attacks.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">16\. Use throttling to manage resource allocation dynamically.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">17\. Use WebSockets for real-time communication.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">18\. Use Heartbeat Mechanisms to detect failures.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">19\. Consider using a message queue for asynchronous communication.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">20\. Implement data partitioning and sharding for large datasets.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">21\. Consider denormalizing databases for read-heavy workloads.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">22\. Consider event-driven architecture for decoupled systems.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">23\. Use bloom filters to check for an item in a large dataset quickly.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">24\. Use CDNs to reduce latency for a global user base.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">25\. Add a caching layer to reduce database load and improve response times.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">26\. Use write-through cache for write-heavy applications.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">27\. Use read-through cache for read-heavy applications.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">28\. Use object storage like S3 for storing large datasets and media files.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">29\. Implement Data Replication and Redundancy to avoid single point of failure.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">30\. Implement Autoscaling to handle traffic spikes smoothly.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">31\. Use Asynchronous processing for background tasks.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">32\. Use batch processing for non-urgent tasks to optimize resources.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">33\. Make operations idempotent to simplify retry logic and error handling.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">34\. Use microservices when suitable for flexibility, scalability, and maintainability.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">35\. Consider using a data lake or data warehouse for analytics and reporting.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">36\. Implement comprehensive logging and monitoring to track the system’s performance and health.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">37\. Implement circuit breakers to prevent a single failing service from bringing down the entire system.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">38\. Implement chaos engineering practices to test system resilience and find vulnerabilities.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">39\. Design for statelessness when possible to improve scalability and simplify architecture.</span>  
<span style="color: rgba(255, 255, 255, 0.9);">40\. Use a publish-subscribe model for real-time updates and notifications.</span>