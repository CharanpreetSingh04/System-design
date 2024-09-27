# System Design

System design involves creating a system that can effectively manage **availability**, **scalability**, and **concurrency** with minimal trade-offs.

---

## Fundamental Topics in System Design:

1. **Database**: The most crucial and fragile part of the system, responsible for storing and managing data.
2. **Caching**: Reduces computation by storing frequently accessed results, improving performance.
3. **Scaling**: Adjusting the number of servers dynamically based on the load to maintain performance.
4. **Concurrency**: Efficiently handling multiple simultaneous requests to prevent bottlenecks.
5. **Delegation**: Offloading tasks to make the server more available to handle new requests.
6. **Communication**: Ensuring smooth communication between end users and servers to keep the system responsive.
---

## **Database**:
The most crucial part of the system, which needs to scale and handle high traffic. The choice of database should align with the specific use case to ensure performance and reliability.

### Common Practices:

**Soft Delete**:
- Instead of permanently deleting data, a "soft delete" marks the entry as deleted but keeps it in the database.

   **Benefits of Soft Delete**:
   1. **Recoverability**: Deleted data can be restored.
   2. **Archival**: Useful for data retention.
   3. **Audit**: Track deletions for auditing purposes.

- Systems like Gmail and Facebook use soft deletes to allow data recovery and avoid performance hits from restructuring the database B+ tree.

**Connection Pool**:
- Pool of TCP connections already set up to use so that DB won't waste time in making one.
---

## **Caching**:
Caching involves storing values in a key-value format to avoid costly computations and improve system efficiency.

### Technical Uses:
- Reduces **disk I/O**, **network I/O**, or **computation** time.
  
### Common Use Case:
- When an API server requests data, it first checks the cache. If the value isn't found, it queries the database.

Tip: cache can be used at any place not just before DB call. A few places where cache is used are
Operating Systems, Networking layers including Content Delivery Networks (CDN) and DNS, web applications, and Databases. 

**Cache Diagram**: ![Cache Design](https://github.com/CharanpreetSingh04/System-design/blob/main/Cache.png)

### Caching Opportunities

1. **API Server Main Memory**
   - Store static content like URLs, API keys, and secrets.
   - **Disadvantages**:
     1. Limited capacity.
     2. API server down -> data unavailable.
     3. Inconsistent for transactional data.

2. **Materialized DB Views**
   - Optimized views that cache query results for faster access.
   - Unlike normal views, data is stored and not re-fetched from the DB.
   - Create via: 
     ```sql
     CREATE MATERIALIZED VIEW mymatview AS ${{DB query}}
     ```
   - Refresh via:
     ```sql
     REFRESH MATERIALIZED VIEW mymatview;
     ```
3. **Browser**
   - Local storage can store JWT tokens, static images, and static assets.

4. **CDN (Content Delivery Network)**
   - Cache full responses (e.g., search results) for quicker retrieval, avoiding calls to the API server.

5. **API Server Disk**
   - Utilize the server's disk space to store data, freeing resources by reducing the need to query the DB server.
   - In many systems, the server disk is underutilized so that extra storage can be leveraged.
     
6. **Load Balancer**
   - Can cache error responses, such as for incorrect URLs or body types, to optimize performance and reduce repeated server hits.

7. **Centralized Cache**
   - The most common use case (illustrated in the diagram at the start of this section).

**Note**: Among all the above use cases, the most feasible option should be selected based on cost and performance considerations.
---


## **Scaling**:
The ability to handle a large number of concurrent requests ensures that the system remains available.

### Types of Scaling:
1. **Vertical Scaling**: Increase server RAM and disk capacity. (HULK)
   - Easier to manage but risks downtime.
   - Single point of failure.

2. **Horizontal Scaling**: Add more machines with the same configurations. (MINIONS)
   - Fault tolerance, harder to manage.
   - Linear performance improvement.
   - Examples: AKS, AWS, etc.
   - **Kubernetes (K8s)** is widely used for horizontal scaling. Below is the architecture of Kubernetes.
     **Link**: ![Kubernetes Diagram](https://github.com/CharanpreetSingh04/System-design/blob/main/minikube.png)
   - **Cons**: 
     1. Complex architecture
     2. Network Partitioning

### Best way to scale:
Scale vertically first then horizontally.
![scaling](https://github.com/user-attachments/assets/54b8efff-d385-4709-961c-a6f86ec27ff6)

Whenever scale, always do it bottoms up.
![image](https://github.com/user-attachments/assets/36463129-7711-4384-b56b-bb4cb6f52633)

### Scaling DB

1. **Vertical Scaling**: Increase the specs of the DB server by adding more RAM, and disk space, and increasing the connection pool size.
![image](https://github.com/user-attachments/assets/41a9c9b0-41fc-41fb-9c49-77820430c9d2)

2. **Read Replicas**: Create replicas of the master DB to handle read-heavy traffic. Writes happen on the master, while replicas handle the reads.
![image](https://github.com/user-attachments/assets/c2d2b12b-8cac-4da1-91e2-2f16ed2aa25b)


3. **Sharding**: Split the database into multiple parts. The API server uses a hash or identifier to decide which shard (database) to interact with. For example:
![image](https://github.com/user-attachments/assets/79897b37-3955-4c29-aa76-c45ed008150d)




---

# System Design References

1. **Online/Offline System**:
   - Detailed design of a system to track user status (online/offline).
   - Includes database schema, API design, and scalability considerations.
   
   **Link**: [Online/Offline System](https://github.com/CharanpreetSingh04/System-design/blob/main/online-offline-system.md)


