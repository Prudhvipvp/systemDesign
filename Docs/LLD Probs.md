

1.  First get clarity about the asked question.
    
2.  What all are the features to be implemented in given app. Search? Post tweets? Post Video? Update tweet ? etc.
    
3.  Now try to have some nfr numbers about the scale of the system. - Capacity Estimation(Bandwidth and storage)
    
4.  Now, have the relevant api interfaces for these features
    
5.  Define Data model of app, the Users table, tweets table etc
    
6.  Have a high level design with this interactions. If its read heavy, then we can have a separate db replicas(write replicas) for write only, and many read replicas
    
7.  Dig deeper based on interviewer feedback on any one component. Based on scale requirements, ask questions like is there any single point of failure, can we cache and if yes at where. What database to use(sql/Nosql)
    
8.  We assume a modern server can store up to 4TB of data (disk) and  has 144GB of memory.
    
9.  While db sharding - think about, what if the index we shard becomes hot and most requests come to 1 shard only.
    
10. A webserver is like the frontend server of your app(that delivers static content) and application server is the backened server(that delivers dynamic content). So the client - web browser thats shows the UI gets this html etc from web server and the data for filling will be fetched from app server.
    
11. At a high-level, our web servers will manage users’ sessions and application servers will handle all the ticket management, storing data in the databases as well as working with the cache servers to process reservations.
    
12. For a push system, once a user has published a post, we can immediately push this post to all the followers. The advantage is that when fetching feed you don’t need to go through your friend’s list and get feeds for each of them. It significantly reduces read operations. To efficiently handle this, users have to maintain a Long Poll request with the server for receiving the updates
    
13. Think of indexing database when its read heavy like in google maps
    
14. Quad Tree - a tree db kind which stores lat, longs in the tree's nodes. Used in situations where u need nearest places to a given lat long. See Yelp design for more clarity.
    
15. The core thing of the service is ticket booking, which means financial transactions. This means that the system should be secure and the database ACID compliant.
    
16. The justification for data sharding is that, after a certain scale point, it is cheaper and more feasible to scale horizontally by adding more machines than to grow it vertically by adding beefier servers.
    
17. In the normalized form, the data is organized into separate tables to avoid redundancy(good sql tables design). In the denormalized form, customer information is duplicated in the `DenormalizedTable`. This can lead to redundancy, but it might be suitable for scenarios where read performance is critical, and you want to avoid joins for common queries.
    
18. Simply saying, an index is a data structure that can be perceived as a table of contents that points us to the location where actual data lives. So when we create an index on a column of a table, we store that column and a pointer to the whole row in the index.  So clearly indexing increases read performance but degrades writes as we have to write to index also along with db
    
19. Consistent hashing is a very useful strategy for distributed caching system and DHTs. It allows us to distribute data across a cluster in such a way that will minimize reorganization when nodes are added or removed. Hence, the caching system will be easier to scale up or scale down.
    
20. In LLD, whenever possible use a List instead of a map
    
21. Also, the repository should be interface and all repos should extend it

Sequential distributed Unique ID Generator -

![Screenshot 2024-01-07 at 6.42.50 PM.png](../_resources/Screenshot%202024-01-07%20at%206.42.50%20PM.png)

**URL Shortening Service -**

1\. The Simple approach of hashing the incoming URL using md5 and converting it to base64 and taking first 6 characters. But it can result in collisions, in which case we have to keep changing the ending characters till we find unique(this requires frequent checks from existing hashs). Also, two URLs with same but encoding in one(%3F inplace of ?) should ideally give same url. So we have to normalise incoming url everytime.

2\. Better and faster way is having a separate Key Generator Service. We will have a KEY-DB which pre generates 64<sup>6  (base64)</sup> unique strings and store them in two tables - Used and Not used tables. The KGS Service keeps loading keys from unused table (and all these keys will be transferred to used table in backend). Now, KGS has all fresh keys to serve whenever asked for. Note, that whenever, key is given by KGS. It needs to acquire a read lock on the datastructure holding the keys and then delete the key from it. Also, the pods can periodically, fetch keys from KGS and store it their cache for faster. We anyways have Billions of keys, so even if a pod/KGS dies, the keys in the cache are lost, but its ok as we can cache again from KEY-DB.

3\. Maybe you can use the above stateless unique generator svc and convert the output int to base64. Note that that svc used 2^64 bits  which convert to 64^10.66 - so around 11 characters of base 64. So either trim the 2^64 to lesser bits(But we have a limit of 41 bits epoch time - you can start from current time instead of 1960 to reduce this size).

&nbsp;

Whenever , there is a scope for async -> Use queue. Even in a chat service, a user receiving a msg can be async. The queue will hit notification svc which will notify the user

&nbsp;

&nbsp;

Parking Lot -

&nbsp;

&nbsp;

FoodKART -

1\. Dont use same object for multi variables. Have separate object for input and when it transforms - use another object. Dont have a field in initial object which remains free for sometime

2\. Always write 1 service at a time to avoid confusion  
<br/>

3\. While creating entities, see if any inheritance can be used. In orders - there generally is a order status cycle. So in such cases, you can use a base Order class

4\. Its okay to duplicate some data.

5\. First decide on what you will be storing permanently. The rest transitive objects can have duplicate fields also as they will be removed from mem anyways.

6\. Verify the inputs and outputs of the each services also

7\. While creating entities only, decide clearly where this will be used in which service. If needed created separate entities for i/p request object and later custom service object etc and define names accordingly

8\. In making services, mind for cyclic dependencies.

1.  **Command Pattern:**
    
    - Command interface and concrete command classes for different user inputs.
        
    - Example: MoveLeftCommand, MoveRightCommand, MoveUpCommand, MoveDownCommand.
        

&nbsp;