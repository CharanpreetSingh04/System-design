# Online/Offline Indicator System Design

## Requirements:
- **Online/Offline Status**: The system must show users as online when they are active and offline when they are inactive.
- **Scalability**: Must handle up to 1 billion users efficiently.

---

## Database Schema Design:

**User Table:**

```table
user_id     | int (primary key)
is_online   | boolean

               ___________
   O           | API      |
  _|_   --->   | Server   |  ---->   Database
  / \          |__________|
  User

Identify the API calls that are required.
1. Get status - 
   a. One way is to make an API call to fetch the status for each user. That way we can consolidate get status responses of all users on the page.
   b. Another way is to make batch API calls to fetch statuses for a batch of users so that we can reduce the number of API calls to the server.
2. Put status -
   a. Pull base model - API server will ping the client in the loop to check if the user is online/offline. 
                        This cannot be possible here as API server can not talk to the front end.
   b. Push base model - Client will push the info that it is alive/unalive.
                        This can be achieved by sending info to the server that the client is alive. 
                        This will happen repeatedly in the interval of threshold time (20 seconds). This is known as heartbeat.
                        If the client does not respond in this interval then we mark that user offline.
                        Instead of is_online, we need to store the last heartbeat which the client has sent to the server.
                        
                        Pulse
                        ------------------------
                        user_id              int
                        last_hb              int
                        Now, whenever heartbeat is received then an update on the database is done which updates the last_hb column with current time in epoch seconds.
                        
                        Update pulse set last_hb = NOW() where user_id = :userId
                        For eg:
                        pulse
                        ------------------------
                        u1                100645
                        u2                100032

While fetching user status, 
1. If user entry does not exist in DB ---> offline
2. If last_hb < NOW() - threshold(20 sec) ---> offline
3. Online

Let's estimate the scale:

100 users -> 100 entries
100000 users -> 100000 entries
1B users -> 1B entries

There are only two columns with sizes of 4-4 bytes each.
So, for 1 B users, data can be in the database can be maximum:
(4 + 4) * (1B) = 8 Billion bytes => 8 GB.

## Storage Optimization
 
### Approach 1: Handling Offline Entries
 
Do we really want to keep entries for users who are offline?
 
The answer is no. We can use the following logic:
 
1. If a user entry is present, the user is online.
2. If the entry does not exist, the user is offline.
 
We only need to expire (delete) entries after the threshold:
 
- **Option A**: Write a cron job that deletes all entries older than 30 seconds. This requires managing an additional service running the cron job.
- **Option B**: Offload this task to a data store like Redis or DynamoDB, which support TTL (Time to Live). When receiving a heartbeat, update the entry in Redis/DynamoDB with a TTL of 20 seconds. This means every heartbeat extends the expiration time by 20 seconds.
 
### Approach 2: Storage Requirements and System Performance
 
Do we need to worry about storage? With 1 billion users, the storage requirement is only 8 GB.
 
However, we should optimize for overall system performance, particularly if handling a high number of requests. For instance, if a heartbeat is sent every 10 seconds, each user sends 6 updates per minute:
 
- For 1 million users: 6 million updates per minute.
- For 6 million updates per minute: 6 million TCP connections are created from the API server to the database.
 
The database must handle this load effectively. Connection pooling can help manage this by reusing existing TCP connections rather than creating new ones for each request. The connection pool size can be adjusted based on the number of connections the database can handle and the number of servers available.
 
**Connection Pool Example**
 
![Connection Pool](https://raw.githubusercontent.com/CharanpreetSingh04/System-design/main/connection-pool.png)