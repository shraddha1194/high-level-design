## Caching

> Caching is idea of

1. Browser / client side cache - Reduces customer faced latency ex. caching previous posts and all when we open instagram
2. App Server cache - Store some data in app server so that compute latency is reduced
3. Global Caching - If all the app server need a common set of data then instead os storing it seperately on each machine, simply store data at a global caching level (cluster of machines), it will still be faster than db layer access cause db layer will read data from disk which will need to be located then loaded and sent back. Consistent hashing can be used on top of Global caching layer.

## CDN - Content Delivery Networks

Big static media content (in MBs/GBs like videos, photos etc.) need to be transferred again and again for every individual user. 
* Netflix / data source is going to be choked.
* Duplicate data transfer of each user (even if they are sitting in same room)
* Latency of data transfer is proportional to distance (from data center/ servers) - data transfers in optical fiber under ocean

Big static - cause video at a particular timestamp does not change, image's particular pixel does not change.

Blob storage - At Db layer we usually store numeric data, for images, profile photo or videos we use specialized compute machines called blob storage or S3 or File storage.

DNS piggyback is missing here, DNS neighbourhood cache.

CDN ex. Akamai, AWS, Facebook
These big companies will have their machines all across the globe. This is called content delivery network.
These machines are in hierarchy.
continent level - country level - state level - region

CDN preemptively copies the data and propagates to various below levels depending where is request is coming from.

Netflix like companies will do agreement with Akamai that I will place my content on your central server. And for my users you will provide the data.

Akamai CDN is not part of Netflix data center.

Netflix Data flow:
1. Client will goto DNS and get IP address (gateway + Lb).
2. Will send request to gateway, fetch data from blob server and send the response.
3. Next time the request comes to fetch the same data, it replies with CDN url.
4. The client then can fetch this content from CDN server in the reply.

This way netflix server is not choked.

What will Akamai do?
* Share the exact name of the resource that is requested
* Share a list of all the servers it has in the region

Netflix will use these details to reply with CDN url.

Anycast - The response is not mapped to any specific machine IP but to a region, CDN machine that region will be able to respond.

CDNs do not get choked because they also do load balancing, is servers in blr are loaded then redirect the load to chennai.

### Cache is just a copy and can go out of sync.

1. Cache is not the ultimate store of data hence it doesn't have the source of truth.
2. Cache is limited in size. We can't cache every data point and data update.

> Cache data can become inconsistent. It may or may not be a big concern depending on the use-case.

View count on instagram being inconsistent is ok.

In IRCTC, number of tickets remaining being inconsistent can cause problems.

## Cache Invalidation Strategy
We should have strategy to kick out the data from cache. The older the data in cache is, more prone to inconsistency it is.

1. TTL - Time to Live : Data resides in cache for limited amount of time. One TTL time is over the data is removed from the cache.

## Strategies used for writing to cache
#### 1. Write through cache - 
>When a data needs to written / saved, first write it in cache then only write in database. This ensure that cache is always in sync with DB.

**CONS**
* The write latency is going to be increased.
* Given your cache size is limited and in this strategy we are trying to write everything to the cache, it will have loads of cache evictions.

**PROS**
* Data is consistent at all times
* If the system is read heavy then this system will speed up your reads

#### 2. Write back cache
>Writes to the cache, then cache responds with Ok and it does not worry aboyt writing to DB. 
Then later cache writes to DB.

**PROS**
* Very fast approach
* The DB can handle writes in batch (it will be lot less loaded)

**CONS**
* Data can get lost forever (Cache restarts and write was not done then data will be lost)
* Data inconsistency

#### 3. Write around cache 
>Client writes on DB and its done. Later based on requirement the cache is loaded (either db pushes in async fashion or cache reads from DB).

**PROS**
* PRO - data is always persisted

**CONS**
* Latency is high, since cache needs to be loaded initially
* data is inconsistent

## Cache Eviction Strategies
1. FIFO - First in first out
2. LRU - Least recently used
3. LIFO - Last in first out
4. MRU - Most Recently used

## Real Life Examples - Rank list in contest
* Content approximately have 10k people, each person will be a client.
* Client is taken to gateway after checking IP from DNS
* We have app servers
* then we have database storage

### How to read the questions? 
Given the question in the contest are only 10, then instead of reading them everytime from DB it is cached by app servers. The load balancer then assigns the requests in round-robin fashion (stateless) to each of the app server.

Case - when user increases from 10k to 120k 
> Add more compute servers and these will load data in the cache and will be ready to take up the load (stateless manner)

### How to read the rank list?
Suppose after the submission of the answers there is some evaluation happening against the test cases, and then it is stored in database. Like first submission was made for question 1 and user got TLE. All of these need to be persisted.

If at a point when request comes to get rank list and at that time we bring all the data from DB to app server and so teh computations and then respond with the result is actually a very bad approach. This will increase load on app server as well as database layer.

Dedicate an app server to perform scheduled processing, it wil lbasically run a scheduled job (CRON job) every 5 min that would compute and generate rank list in every 5 mins. Once rank list is generated put it in global cache. Any app server which receives a request will fetch the list from cache and respond. 
This reduces latency significantly. 

Once a student answers the questions or is writing a code every 1 min there are a call to save the code so that even when we refresh the code is not lost. 

### Code evaluation
The tests case file is stored in file storage since it is huge.
If we store in global cache - reading file again and again is not feasible.
Send test case files to client, to run at their end - Client should not run the test cases.
Can we use CDN - No it is used by client to fetch something

Can we store it in app server - 
These app servers need to stateful and make them stateful wrt to problem_id

Based on problem we are submitting we can go to a particular app server. In this case one app server needs to cache questions related to only one app server.


