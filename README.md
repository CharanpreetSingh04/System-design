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

## **Database**: The most crucial part of the system, which needs to scale and handle high traffic. The choice of database should align with the specific use case to ensure performance and reliability.

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



# System Design References

1. **Online/Offline System**:
   - Detailed design of a system to track user status (online/offline).
   - Includes database schema, API design, and scalability considerations.
   
   **Link**: [Online/Offline System on GitHub](https://github.com/CharanpreetSingh04/System-design/blob/main/online-offline-system.md)


