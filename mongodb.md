# What I Learned Today: MongoDB Basics

Today, I explored the fundamentals of MongoDB, its differences from relational databases, and how it handles data. Here's a summary of what I learned:

---

## 1. MongoDB Overview
MongoDB is a **NoSQL database** that stores data in a **document-oriented** format. Unlike traditional relational databases like MySQL, MongoDB is schema-less, allowing for flexible and dynamic data structures.

### Key Features:
- **Document-Oriented:** Stores data in JSON-like documents.
- **Schema-Less:** Documents in the same collection can have different structures.
- **High Scalability:** Supports horizontal scaling through sharding.
- **Flexible Querying:** Provides a powerful query language to interact with data.

---

## 2. Collection vs. Table
In MongoDB, data is stored in **collections**, which are conceptually similar to **tables** in relational databases, but with key differences.

| **Aspect**                | **MySQL Table**                                    | **MongoDB Collection**                             |
|---------------------------|----------------------------------------------------|---------------------------------------------------|
| **Schema**                | Fixed schema (predefined columns and types).       | Schema-less (documents can vary in structure).    |
| **Rows vs. Documents**    | Data is stored in rows, with the same columns.      | Data is stored in JSON-like documents (BSON).     |
| **Data Relationships**    | Strongly relational, using JOINs.                  | Typically embeds related data or uses references. |
| **Scalability**           | Vertical scaling (add more hardware).              | Horizontal scaling (distributed systems).         |

---

## 3. Collections and Documents

### What is a Collection?
A **collection** is a grouping of related documents in MongoDB. It's similar to a table in MySQL but does not enforce a fixed schema.

### What is a Document?
A **document** is a JSON-like object that represents a single record in a collection. Documents can have:
- Dynamic fields (no fixed columns).
- Nested structures (embedded objects or arrays).

### Example:
#### MongoDB Document:
```json
{
  "comment_id": "123",
  "content": "This is a comment.",
  "createdBy": "user_456",
  "createdAt": "2025-01-06T12:00:00",
  "isDeleted": false
}
```
## 4. Querying Data in MongoDB

MongoDB uses a powerful query language for filtering and manipulating documents.

Example Query:

MySQL Query:
```json
SELECT * FROM comments WHERE post_id = 123;
```

MongoDB Query:
```json
db.comments.find({ postId: 123 });
```
## 5. Indexing in MongoDB

Indexes improve query performance in MongoDB. I learned how to create indexes for specific fields to speed up queries.

Example Index:
```json
db.comments.createIndex({ postId: 1 });
```
This creates an index on the postId field to make filtering faster.

## 6. Single vs. Multiple Collections

Single Collection Approach:

You can store related data (e.g., feed comments and livestream comments) in the same collection by adding a distinguishing field like contextType or livestreamId.

Example:
```json
{
  "comment_id": "123",
  "content": "Nice post!",
  "contextType": "feed",
  "feedId": "feed_456",
  "createdBy": "user_123"
}
```
```json
{
  "comment_id": "456",
  "content": "Amazing livestream!",
  "contextType": "livestream",
  "livestreamId": "live_789",
  "createdBy": "user_789"
}
```
## 7. Key Takeaways
	1.	Collections are like tables, but more flexible and schema-less.
	2.	MongoDB supports dynamic data structures with JSON-like documents.
	3.	Using indexes is crucial for optimizing query performance.
	4.	Carefully design your schema based on your application’s needs:
	•	Use a single collection for related data to simplify logic.
	•	Use multiple collections if separation or unique functionality is required.
	5.	MongoDB provides flexibility and scalability for modern applications, making it ideal for dynamic or high-traffic systems like comments on feeds and livestreams.
