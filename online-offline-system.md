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

# API Design for Online/Offline Indicator System

## 1. **Get Status API**

### a. **Single User Request**
- **Description**: Fetch the online/offline status of an individual user.
- **API Call**: `GET /status/{user_id}`
  - **Input**: `user_id`
  - **Output**: `{ "user_id": user_id, "is_online": boolean }`

- **Use Case**: This approach can be used when the client needs the status of a single user, such as on a profile page.

---

### b. **Batch User Request**
- **Description**: Fetch the online/offline status of multiple users in a single API call.
- **API Call**: `POST /status/batch`
  - **Input**: `[{ "user_id": userId1 }, { "user_id": userId2 }, ...]`
  - **Output**: `[ { "user_id": userId1, "is_online": boolean }, { "user_id": userId2, "is_online": boolean }, ... ]`

- **Use Case**: This approach reduces the number of API calls by fetching the status for multiple users at once, ideal for pages showing a list of users (e.g., messaging apps or friends lists).

---

## 2. **Put Status API**

### a. **Pull-Based Model** (Not Applicable)
- **Description**: The API server would regularly ping clients to check if they are online. However, this model is not feasible, as the server cannot initiate communication with clients.

---

### b. **Push-Based Model** (Preferred)
- **Description**: The client periodically sends a "heartbeat" to the server, indicating that the user is online.
- **API Call**: `POST /status/update`
  - **Input**: `{ "user_id": userId, "timestamp": epoch_time }`
  - **Output**: `{ "status": "success" }`

- **Use Case**: The client sends this request at regular intervals (e.g., every 20 seconds). If the server does not receive a heartbeat within the threshold time, the user is marked as offline.

---

## Heartbeat Database Schema (Push-Based Model):

**Pulse Table:**

```table
user_id     | int (primary key)
last_hb     | int (epoch timestamp)
                        
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
