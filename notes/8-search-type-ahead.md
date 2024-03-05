# Search Type Ahead

## What is search type ahead?
Google search gives us suggestions as it start trying.

Searching product in amazon - auto suggests what we are looking for.

It's a feature that helps us with suggestions ahead of what we're typing.

### 1. Minimum Viable Product (MVP)
* On writing prefix of a query we get suggestions from system - assume we get 5 suggestions
* Suggestion we get should not be random suggestions - should be based on popularity
* At every keystroke after first 3 chars the suggestions should be provided.

### 2. Estimation of scale
Build a system at Google scale (large scale)

* How many users do you think Goggle have?
    > total - 4 billion users
    > 
    > daily active users  - around 1 billion
    
* How many queries are each user on avg going to make?
    > Queries / day ~ 10 per user
    > 
    > in total for 1 billion users ~ 10 billion queries per day

    > For each search we are going to send multiple queries to backend to get the suggestions.
    >  
    > If we select no further queries to the backend.
    >  
    > If we type next char we are going to make the next query to BE to get next set of suggestions.

* How many queries do we need to send to type ahead system for each search?
    > On avg assume it takes 5 queries
    >
    > In total, it will take 10 billion * 5 = 50 billion type ahead queries / day to our system.
    
* Do we need to write data in our system?
    > Yes, because data will not automatically be created for suggestions. 
   
* How many read queries / write queries will we get?
    
    > While searching we are getting 50 billion queries - these will be read queries
    >   
    > The data is written to DB only on clicking of enter or search button click meaning we have ~ 10 billion writes
    
* Is the system read heavy or write heavy?
    
    > Number of read queries = 5 * Number of write queries
    >
    > Unless the number is 100 times, read will not overpower write
    >
    > Hence, it is read + write heavy system. 
    
* Concurrency - QPS - Queries per second
    > 1 day - 10 billion query
    >
    > 10 billion / 24 * 60 * 60 = 1.1 * 10 ^ 5 queries / second
    >
    > approx 100K write QPS
    >
    > approx 500K read QPS 
    >
    > QPS helps to decide how many app serves are required in our cluster to handle the requests.
    
* How much storage is required?
    >  updating the popularity / frequency will never create a new data.
    >
    >  extra space is required only for the new words new searches 
    >
    > As per google stats, every day 10% of the queries are roughly new. 
    > As per our data - 1 billion new searches / day
    >
    > Suppose we have 30 chars per search ~ approx 30 bytes (chars) + 10 bytes (count) + 60 bytes (metadata) = 100 bytes
    >
    > 1 billion new searches * 100 bytes = 100GB data / day
    >
    > For a system over 10 years for active data storage: 
    > 100GB * 365 * 10 = 1TB * 365 = 365TB data in 10 years
    >
    > In 1 master and 3 slaves replication we wil have 365TB * 4 = 1500TB
    >
    > Sharding is a necessity with this huge data. 
 
### Design Tradeoffs
* Availability > Consistency
* Latency required is very low - the system competes with teh typing speed of the users and hence the latency should be super low.

### Design deep dive

1. List all the APIs:
   * getSuggestions(prefix_string, limit = 5)
   * updateFrequency(search_query) - will be called by client, by a parallel internal system (google search would also update the frequency)

2. Components:
    * Client - will make the request
    * Client will make request to DNS to get the IP
    * Then makes call to our Gateway / Load balancer.
    * App server
    * Database layers
    * Sharded database - master slaves and hence it needs load balancer / reverse proxy

3. What we have to store database?
* We need to store string and frequency.
    * SQL - Fixed row size drawback (query - size will depend on user input, wastage of space)
    * NoSQL
    * Trie vs HashMap - 
      > Trie approach:
      > 
      >Database cannot fit in one machine, and we need to shard, but how to shard is a challenge.
      >    
      > Query on Trie might lead to multiple shard query which is also bad.
      > 
      > Backtracking across shards to find all the suggestion across shards and then sorting on frequency is also difficult.
      
      > HashMap approach:
      > 
      > DB1 cluster - updateFrequency action
      > 
      > search_string, frequency
      > 
      > search query by frequency, shard key is search query
      > 
      > ------------
      > 
      > DB2 cluster - getSuggestions
      > 
      > Prefix and top 5 suggestions
      >
      > But how to maintain top 5 suggestions for each prefix - how DB gets its updates?
      > 
      > Once frequency increases in DB1, then this needs to be propagated in DB2
      > 
      > To do this there are two ways - messaging queues (TBD) or threshold approach.
      > 
      > Threshold Approach - only when my frequency crosses a set threshold value we go and update DB2. Since, consistency is not that important we do not need immediate update. For every 10k, 100k update in DB1 we update DB2, this reduces write factor by 10k and 100k respectively.
      > 
      > Choosing periodicity over threshold is bad idea because we may have events like elections where search rates are very high, but updating once in a week of so might not help here.
      >
    
      > **Federation of database** 
      > 
      > Using multiple database clusters within a system to handle different scenarios is called database federation.

    * Caching swad anusar- 
      * client side optimization (CSO) - to reduce latency
      * global caching for popular queries
      * db lever caching
    * Recency bias Factor