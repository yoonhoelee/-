# Managing Reactions with Redis

This post details how to use **Redis** for managing and querying reactions (e.g., likes/dislikes) efficiently in a live-streaming application. We'll cover key concepts, commands, implementation details, and best practices.

---

## **Why Redis for Reactions?**

Redis is a high-performance, in-memory data store, making it ideal for use cases requiring:
- **Real-time updates**: Tracking live reactions efficiently.
- **Scalability**: Handling high-concurrency workloads.
- **Low latency**: Providing instant feedback to users.

Using Redis for reactions enables:
- Fast reads and writes.
- Dynamic data manipulation.
- Seamless integration with databases for hybrid architectures.

---

## **Key Design**

### **Redis Key Structure**

1. **Reactions by Replay**:
   - **Key**: `reaction:replay:{replayId}`
   - **Value**: A hash where:
     - Each field is a `userId`.
     - The value is `1` if the user has reacted (liked).

   Example:
   ```
   Key: reaction:replay:1
    Fields:
      user123: 1
      user456: 1
      user789: 1


2. **Expiration**:
- Keys can have an expiration time (e.g., 90 days) to optimize memory usage and ensure stale data is cleaned up.

---

## **Redis Commands**

### **1. Add or Update a Reaction**
```bash
HSET reaction:replay:{replayId} {userId} 1
```
•	Result: Adds a reaction for user123 to replayId = 1.

### **2. Remove a Reaction**
```
HDEL reaction:replay:{replayId} {userId}
```
### **3. Check If a User Reactedn**
```
HEXISTS reaction:replay:{replayId} {userId}
```

•	Result:
	•	1: The user has reacted.
	•	0: The user has not reacted.
 
## Java Implementation

Toggle Reaction Logic

This method toggles a reaction for a given user and replay.

``` java
public boolean toggleReaction(Long replayId, String userId) {
    String replayKey = RedisConstants.generateReplayReactionKey(replayId);

    // Check if the user has already reacted
    boolean hasReacted = redisRepository.hasUserReacted(replayKey, userId);

    if (hasReacted) {
        // Remove the reaction (dislike)
        redisRepository.setReaction(replayKey, userId, false);
        return false; // Reaction removed
    } else {
        // Add a reaction (like)
        redisRepository.setReaction(replayKey, userId, true);
        return true; // Reaction added
    }
}
```

Check If a User Reacted

This method checks if a specific user has reacted to a replay.
```
public boolean isReacted(Long replayId, String userId) {
    String replayKey = RedisConstants.generateReplayReactionKey(replayId);
    return redisRepository.hasUserReacted(replayKey, userId);
}
```
Get Total Reaction Count

This method retrieves the total number of reactions for a replay.
```
public Long getReactionCount(Long replayId) {
    String replayKey = RedisConstants.generateReplayReactionKey(replayId);
    return redisRepository.getReactionCount(replayKey);
}
```
## Best Practices
1.	Use Expiration:
	•	Set an expiration for reaction keys to clean up stale data.
2.	Monitor Redis Memory:
	•	Regularly monitor Redis memory usage, especially with high traffic.
3.	Hybrid Storage:
	•	Combine Redis with a database for persistent storage if needed.
4.	Error Handling:
	•	Handle Redis failures gracefully and provide a fallback mechanism.
