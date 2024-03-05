## Sharding

Everytime we face load on a system, we either scale it vertically or horizontally. Same applies to database as well. One machine will never be able to handle huge data, imagine facebook handling all the data on one machine.

> Adding a new machine with better configuration is vertical scaling.

> Adding number of machines of same configuration to handle and distribute the load across the machines is called horizontal scaling.

> Sharding means if we cannot a fit a data on one server then divide it in mutually-exclusive collectively exhaustive pieces. Each piece is called shard.

> Mutually-exclusive - One data point is present only on one machine.

> Collectively-exhaustive - meaning it's not possible that a data point is missed.

Each shard is divided between multiple machines. A shard is set of multiple machines.

## How would Database look like if we shard SQL database?

If facebook used SQl database we would have the following tables:
User - id, name, email, gender, last_seen
User_Friends - user_is, friend_user_id (bidirectional relations are maintained)
Posts - post_id, text, userId, timestamp, content

### How would we shard the data?

> We need shard key to shard the data. Also known as request id, hashing key or partition key.

Suppose we have 4 machines -
1. Users, user_Friends, Posts
2. Users, user_Friends, Posts
3. Users, user_Friends, Posts
4. Users, user_Friends, Posts

It looks like all the machines have same data, mostly looks like replication, but it's not the case. All the server use same template but have different data stored.

> Consistent hashing helps us assign data to each sharding


#### If we shard on user_id:
All the data for user with id 100 will reside on one shard/machine. One machine will have it's user details, post and user-friends on single machine.

### INTRA SHARD Query
>When we piggyback on a single shard to answer a query it is called intra shard query. It is very optimized, since we are able to answer all the queries from one shard.

### INTER SHARD Query
Example when we want to see my feed on facebook, we usually see posts from others.
* To find my friend, I can find my friends on one shard.
* All the posts from different users is on different shards. So goto all the different shards and find their posts.
* App server will collate the data and show the feed.

> When to answer a query we need to goto multiple shards then its known as inter shard query.

> We need to choose the right sharding key so that majority of our frequent queries can be resolved as intra shard queries and only a few of the queries can be handled as inter shard.

If a request goes to:

|1 shard | 2 shard | All shard|
|------|--------|----------|
|Best|Good enough| Worst|

### How is facebook newsfeed build?
* Newsfeed - has posts by friends or the pages that you follow.
* Profile page - Posts that you made.

**Assumptions** - 
* Facebook uses SQL databases

* Following are the tables:
User - id, name, email, gender, last_seen
User_Friends - user_is, friend_user_id (bidirectional relations are maintained)
Posts - post_id, text, userId, timestamp, content

* All data of facebook can fit inside the same machine.

| Profile Page                                                                                     | Newsfeed |
|--------------------------------------------------------------------------------------------------|----------|
| **Posts made by you(user_id X)** - select * from posts p where p.user_id = X LIMIT 10 offset 50; | **Posts made by the friends of ther user with user_id X** - select * from user_friends f join posts p on f.user_id = X and f.friend_id = p.user_id and p.timestamp < now() - 30days LIMIT 10 OFFSET 0; |


We use pagination to limit the data we sho won page and hence limit we don't want to show all 1000 posts on one page and offset to loa dthe data for second, third and subsequent pages.

Reality - All the data can't fit inside single machine.

**User_id as Sharding key** 
This decision means the following:
* user-attributes
* list of friends
* posts made 

All of these will be on same machine

>All the profile page requests will be intra shard and all the newsfeed requests will be interred even worse because it will goto all the shards.
It seems the basic requirement of the facebook newsfeed is not possible this way cause latency will be very high since it required too many queries. We need new approach.


#### How to solve the problem of going to all the shards to create newsfeed?
In any case if we need to reduce the number of accesses to database, we try to cache the data.
But what type of caches?

**1. Can we use CDN?**

Caching a user's newsfeed on CDN is bad idea, cause a user's newsfeed is personal to him and not applicable for other people.
Persisting a person's newsfeed all across globe is a bad idea.

> Note - that the newsfeed will keep on changing, even if we cache the newsfeed it will update after 5 mins as and when a friend makes a post.

Given that data is going to change how are we going to cache the content?

**2. Can we cache newsfeed on client side?**

Of course, we should use client side caching as it will reduce the perceived latency at the client's end but in reality 
user will not see any new post with it. It is good to use it but its not the complete solution in itself.

**3. Can we store everybody's newsfeed in app server cache?**

Let's say the app servers are stateful meaning requests from one user will goto a particular a app server.
We may cache the user newsfeed here, if cache uses LRU then it will mostly have data for the users that are frequent.
The fan factor of the app server cache will be very high.

As soon as a friend or page makes a post the cache of all the related users need to be updated.
If a celebrity makes a post then cache needs to be updated for millions of followers and fan-out factor is very high.
One post is causing too many inter shard queries.

> **Computed cache** - None of the underlying tables had newsfeed data but we are computing the newfeed using multiple tables and whatever is the result we are caching it. This is the idea of computed cache.

> **Denormalization** - 
> 
> In database wr learned about normalization to reduce data redundancy. This is a great theoretical concept but in reality
> data is spread over so many machines that we would require dto goto multiple shards for updates.
> 
> At times, we need to duplicate the data to make sure that the requests land on the same shard.
> 
> In case of user_friends table we are storing bidirectional relations, so data of one user is on one shard.

