#### Clarifying Questions - Facts & Figures
* should the short URL expire? - *live for 100 years.*
* allow custom URLs? - *yes, up to 16 characters.*
* How many shortens/month? - *100 mil/month*
* **200 Reads: 1 Write** ( for every one short URL created, roughly 200 short URL lookups/clicked) - ***Read-heavy.***
* 100 mil short URLs created / month -> 40 short URLs created/sec. -> 8000 reads/sec.
* 100 years storage for 100 mil URLs  - *120 billion objects*. `{longURL, shortURL, $createdAt, userID`. 
* Each object 500 bytes. -> **60TB storage**.
* 700 million reads/day. *Cache 20% of requests* (Pareto Principle - 80% of reads are for 20% of the data). -> **70 GB cache memory storage.**.
####  *Functional Requirements*
* shorten a URL
* clicking on the short URL redirects to long URL
* short URL as small as possible - 7 characters.
* can create a custom short URL - max length 16 chars.
* short URL stays in system for 100 years.
####  *Non-Functional Requirements*
* no downtime.
* URL redirect should be fast.
* app should be resilient to load spikes for both URL generation (Write) and redirection (Read). - load distribution
* servers should expose REST API endpoints for developers.
####  *Server Requirements*
* unique short URL (one-to-one). NO cases of many-one/many-many can be allowed.
	* can be implemented by -
		* base62 encode
		* key generation service - *MD5 security concerns - Collision Attacks.* 
	* Dedicated Database to store and mark already generated random 7 character strings generated using the key generation service.
***
### Base Skeleton Diagram
![[Pasted image 20240404230431.png]]

*Issues in current implementation*:
* does not handle error scenarios or edge cases.
* having one single instance of web server. **Single Point of Failure.**
* Not Scalable.
* Single Database not enough for 60TB of Storage or 8000 reads/sec.

***Improvements***
* Add **load-balancer** between client and server.
* Increase amount of **replicas** of server. protects against DDoS(*Distributed Denial-of-service*) attacks. Adding **Server Redundancy**.
* **Shard** the database.
* Add a **caching** system to - *reduce load on database* & *serve hot URLs faster.*
### Scalable Solution Diagram
![[Pasted image 20240329102442.png]]
***
### Rest API Endpoints

#### Create a Short URL
```
POST /create
create(long_url, api_key, custom_url=RANDOM_HASH_GENERATED_URL)
req body { longurl: long_url, custom: custom_url }

// api_key passed via header or cookie
returns: status or error code, short_url in data
```
#### Lookup a Short URL
```
GET /{short_url}
req body { shorturl: short_url }

// api_key passed via header or cookie
retuns: http redirect response (302) or error code, and original long URL
```
***
### Database
#### Schemas
* **USER**:
	* ID (`PRIMARY KEY`)
	* name
	* email
	* creation timestamp
* **LINK**:
	* Short URL (7 Characters, `PRIMARY KEY`)
	* original URL
	* userID (`FOREIGN KEY`)
	* creation timestamp
#### Architecture
*  **SQL** - Efficient to check if a URL exists in the DB and handle Concurrent Writes. difficult to scale. ACID complaint ( a db update will fully commit or rollback.)
* **NoSQL** - Easier to scale, but *Eventually Consistent* - meaning, change is not guaranteed to propagate throughout the system immediately. *Possibility of  a READ, immediately following a WRITE* may return stale data.

*Choosing NoSQL,*:
* easier scaling for huge storage of 60 TB.
* high reads/writes (8000/s, 40/s)
* Though can be scaled with a relational database with *custom partitioning* & *replication* - complex to develop and maintain. 
#### Sharding
 * use the hashed URL as a **shard key** to determine the bucket.
#### Caching
* system should give priority to hot links - **faster reads**.
* memcached / redis
* a modern day enterprise level server has 64-256 GB of memory. (1-2). Good to add redundancy.
* Cache Replacement - LRU (Least Recently Used), store using *linked hashmap*. linked list components preservers order.