## Social Network Platform - Twitter
***
### Requirements

* **Functional Requirements**:
	1. Follow other users 
	2. Create tweets (Max 140 characters, images & videos)
	3. View feed (Multiple parameters, ranking, algorithms) - *assuming you see tweets only from accounts you follow*

* **Non-Functional Requirements**:
	1.  **Users** - 500M, DAU(Daily Active Users)- 200 M. 
	2. **Read-heavy**: 200M * 100 Reads - 20B Reads/day. (140B of characters + Metadata).  *Assuming 1KB(text) or 1MB(w/ media) of* Reads/day. ~20 Petabytes(PB) of data. ***Eventually Consistent Storage.***
	3. Write: 50M
***
### High-Level Design

![[Pasted image 20240329144900.png]]

***
### API endpoints


```
POST 
createTweet(text, media)
```

Add CreatedAt, tweetid UserName in server-side, http header


```
GET feed(u_id)
```

```
POST
follow()
```
***
### Database
#### Schemas
1. Tweet
	* TWEET_ID(*PRIMARY_KEY*)
	* Content
	* Media (Optional) - *Reference to media.*
	* Username
	* $createdAt
2. User
	* Followers
	* Following

#### Scaling
* Read-Only Replicas (Leader Follower)