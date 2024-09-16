Design online/offline indicator system

Always note all the requirements first.
   a. Show users as online when they are online and offline when the user is not online.
   b. System can be able to handle 1 Billion users in total.

Design of DB schema:

User
------------------------
user_id              int
is_online        boolean

The access pattern of this problem is, that the client will send the user_id and the server has to send the status (small key value read).
When we know that it will be a small read/write then we see that we don't need a disc with a higher memory.
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

How can we do better on storage?

Approach 1: Do we really want entries of the users who are offline?

            The answer is no as we can set our logic like this,
            1. If the user entry is present user is online
            2. If it doesn't user is offline
            
            We only have to expire (delete) entries after the threshold.
            a. One way is to write a cron job that deletes all entries are 30 seconds older than the current time. 
               But then we have to manage another service which has the cron job running inside.
            b. Another way is that we can offload this to our data store. We can use data stores like Redis, DynamoDB, etc.
               These datastores have TTL limits and we can set them according to our requirements.
               Upon receiving a heartbeat update entry in Redis/DynamoDB with Time to live (TTL) as 20 secs.
               This means that every heartbeat moves the expiration time forward by 20 secs.

Approach 2: Do we need to worry about the storage?
            No, As the storage required for 1 Billion users is only 8 GB.
            Then what can we optimise to improve overall system performance?
            A thing to note here is that we are doing only small read/write.
            If there are a lot of requests then we should worry about DB being able to handle concurrent requests and load.
            Suppose, Heartbeat is sent in every 10 secs that means for every user it's sent 6 times.
            For 1M users -> 6M times/min.
            For 6M updates/min -> 6M TCP connections will be created from API server to DB.
            The database should handle that load. This can be achieved by connection pool as in tweaking the min pool size and max pool size.
            Connection pool is the pool of TCP connections already created so that if any request comes than it doesn't have to waste time in creating one.
            The value of max pool size can be defined as:
                  Number of connections DB can handle/Number of servers.



            ![connection-pool](https://github.com/CharanpreetSingh04/System-design/blob/main/connection-pool.png)

