## Decoupling of App server nad Db

In LoadBalancer, we learned that when we type something on the browser( or search for a website specifically), 
the first thing browser has to do is talk to a DNS and figure out which IP address the browser should be talking to, 
and it communicates with the machine at that particular IP. 
And when you go from one machine to multiple machines, you need a load balancer to distribute traffic uniformly, 
but after a point when the amount of information cannot fit into a single machine, then the model needs to shard. 
It can be done using consistent hashing.
However, the machines had both the code and storage in the previous model. _Do you think it is a good model_????

![img.png](img.png)

1. Code and database are tightly coupled, and code deployments cause unavailability.
2. Fewer resources are available for the code since the database will also use some of the resources.

So it is better to decouple code and storage. 
However, the only downside of decoupling is the additional latency of going from one machine to another (code to the database).

So it can be concluded that it is not ideal for storing code and database on the same machine. 
The approach is to separate the code and storage parts to increase efficiency.

![img_1.png](img_1.png)

Different machines storing the same code running simultaneously are called Application Server Machines or App Servers. 
Since they don't store data and only have code parts, they are stateless and easily scalable machines.


------------------------------------------------------------------------------------------------------------------------

## Caching

**The process of storing things closer to you or in a faster storage system so that you can get them very fast can be termed caching.**
Caching happens at different places. 
### 1. In- Browser Caching
    We can cache some IPs so that browser doesn't need to communicate with the DNS server every time to get the same IP address. 
    This caching is done of smaller entries that are likely not to change very often and is called in-browser caching. 
    Browser caches DNS and static information like images, videos, and JavaScript files. 
    This is why a website takes time to load for the first time but loads quickly because the browser caches the information.


![img_2.png](img_2.png)

### 2. CDN ( Content Delivery Network)

You have the browser in some region(say India), and you must fetch the files from the servers located in another region(say the US). 

    When you try to access from your browser -> a request is made to the load balancer -> application server ->  file storage. 

    You know that transferring files and other data will be fast for the machines in the same region. 
    But it can take time for machines located on different continents.

From the website perspective, users worldwide should have a good experience, and these separate regions act as a hindrance. 
So what’s the solution?

The solution for the problem is _**CDN, Continent Delivery Network**_. Examples of CDN are companies like

1. Akamai
2. Cloudflare
3. CloudFront by Amazon
4. Fastly

These companies' primary job is to have machines worldwide, in every region. 
They store your data, distribute it to all the regions, 
and provide different CDN links to access data in a particular region. 

Suppose you are requesting data from the US region. 
Obviously, you can receive the HTML part/ code part quickly since it is much smaller than the multimedia images. 
For multimedia, you will get CDN links to files of your nearest region. 
Accessing these files from the nearest region happens at a much larger pace. 

**Also, you pay per use for using these CDN services.**

![img_3.png](img_3.png)

![img_4.png](img_4.png)

One question : how your machine talks to the nearest region only(gets its IP, not of some machine located in another region), 
when CDN has links for all the regions. Well, this happens in two ways:

1. A lot of ISP have CDN integrations. Tight coupling with them helps in giving access to the nearest IP address. For example, Netflix’s CDN does that.
2. Anycast (https://www.cloudflare.com/en-gb/learning/cdn/glossary/anycast-network/)

This CDN process to get information from the nearest machine is also a form of caching.


### 3. Local Caching
It is caching done on the application server so that we don't have to hit the database repeatedly to access data.

![img_5.png](img_5.png)

![img_6.png](img_6.png)

### 4. Global Caching

This is also termed In-memory caching.
In practice, systems like Redis and Memcache help to fetch actual or derived kinds of data quickly.

![img_7.png](img_7.png)


### Problems Caching brings
1. Cache has limited size
2. It is not the actual source of truth
3. It just stores the replica of original data source (database)
4. Data becomes stale and inconsistent with time -> we constantly need to fetch data after certain interval of time(TTL)
5. Because of limited size, we need to explore eviction policy


![img_8.png](img_8.png)

### Preventing problems : Caching Invalidation Strategy
One solution that is proposed so that cache doesn’t become stale is : **TTL (Time to Live)**
This strategy can be used if there is no problem with the cache being invalid for a very short time, 
so you can have a periodic refresh. Entries in the cache will be valid for only a period. 
And after that, to again get the entries, you need to fetch them again.
So, for example, if you cache an entry X at timestamp T with TTL of 60 seconds, 
then for all requests asking for entry X within 60 seconds of T, you read directly from cache. 
When you go asking for entry X at timestamp T+61, the entry X is gone and you need to fetch again.

![img_9.png](img_9.png)


### Keeping cache and DB in sync: 
This can be done by the strategies like Write through cache, Write back cache, or Write around the cache.


![img_10.png](img_10.png)

#### Write through cache: 
Anything to be written is database passes from cache first(there can be multiple cache machines), storing it (updating cache), and then updating it to the database and returning success. 
If failed, changes will be reverted in the cache.
It makes the writing slower but reads much faster. For a read-heavy system, this could be a great approach.

![img_11.png](img_11.png)

#### Write back cache: 
First, write is written in the cache. 

The moment write in the cache succeeds, you return success to the client. 
Data is then synced to the database asynchronously (without blocking current ongoing request).
The method is preferred where you don't care about the data loss immediately, like in an analytic system where exact data in the DB doesn't matter, 
and analytical trends analysis won't be affected if we lose data or two. 
It is inconsistent, but it will give very high throughput and very low latency.

![img_12.png](img_12.png)

#### Write around cache: 
Here, the writes are done directly in the database, and the cache might be out of sync with the database. 
Hence we can use TTL or any similar mechanism to fetch the data from the database to cache to sync with it.

![img_13.png](img_13.png)


Now let’s talk about the second question: How can you add entries if the cache is full?

Well, for this, you be using an eviction strategy.

### Cache eviction:

![img_16.png](img_16.png)

There are various eviction strategies to remove data from the cache to make space for new writes. Some of them are:
FIFO (First In, First Out)

![img_14.png](img_14.png)

LRU (Least Recently Used):

![img_15.png](img_15.png)


LIFO (Last In, First Out)
MRU (Most Recently Used)

The eviction strategy must be chosen based on the data that is more likely to be accessed. The caching strategy should be designed in such a way that you have a lot of cache hits than a cache miss.



![img_17.png](img_17.png)