# **📌 Scenario-Based Cassandra Interview Questions**

1. **You are tasked with designing a music streaming service using Cassandra. How would you model the data to efficiently store user playlists and song metadata?**  

2. **In a distributed blog platform, how would you design a schema to efficiently handle the retrieval of the latest blog posts from a particular user in Cassandra?**  

3. **How would you design your Cassandra data model for a real-time chat application to ensure all messages are delivered to all participants, and older chats are retrieved quickly?**  

4. **If you're designing a ride-sharing application like Uber or Lyft, how would you leverage Cassandra for handling real-time locations of millions of riders and drivers?**  

5. **You are designing a Cassandra database for a large e-commerce website. How would you create the data model to support operations such as listing all the products in a certain category, keeping track of users' shopping carts, and storing users' order history?**  

6. **In a gaming application, player profiles are read frequently and updated occasionally, but each player’s profile could be quite large. How would you design the data model in Cassandra to support this use case efficiently?**  

7. **You are working with a social networking site and you need to create a feature to retrieve the posts from friends that a user follows. How would you model your data in Cassandra to efficiently support this operation?**  

8. **You have a Cassandra cluster with multiple data centers. How would you configure the replication strategy to ensure low-latency reads and writes, as well as redundancy across different geographic regions?**  

9. **If you are developing a stock market application that needs to store and retrieve real-time and historical stock prices, how would you model the data in Cassandra?**  

10. **Given the task of developing a Cassandra-based backend for an IoT application that constantly receives huge amounts of sensor data, how would you design the data model and which compaction strategy would you choose to handle this use case?**  

11. **Consider a system where you need to track user events on a website and then display a timeline of these events to the user. How would you design the Cassandra tables to efficiently handle this use case?**  

12. **Suppose you have to implement a distributed task queue with Apache Cassandra where tasks have different priorities. How would you design this?**  

13. **Imagine you're developing a movie recommendation system. How would you design a Cassandra schema to efficiently store and retrieve user ratings for various movies?**  

14. **You are tasked with developing an online collaborative document editing platform. How would you leverage Cassandra to store document data and handle simultaneous edits from multiple users?**  

15. **You have a requirement to store time-series data for an application monitoring system, where you need to track system metrics every minute. How would you model your data in Cassandra to serve this requirement?**  

---
<br/>
<br/>
<br/>

# **1. Designing a Music Streaming Service Using Cassandra🔥🔥**  

### **🔹 Key Requirements:**  
✔ **Efficiently store and retrieve song metadata.**  
✔ **Allow users to create, modify, and access playlists quickly.**  
✔ **Support searching by artist, genre, and song title.**  
✔ **Scale to millions of users and songs.**  

---

## **📌 1. Table: Storing Song Metadata**  
🔹 _Stores essential details for each song._  

```cql
CREATE TABLE songs (
    song_id UUID PRIMARY KEY,  
    title text,  
    artist text,  
    album text,  
    duration int,  -- in seconds  
    genre text,  
    release_year int
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `song_id` → Ensures each song has a unique identifier.  
✔ **Stores metadata like artist, album, and genre** for searchability.  

### **📌 Query Examples:**  
✔ **Retrieve a Song's Metadata**  
```cql
SELECT * FROM songs WHERE song_id = 5678;
```

✔ **Insert a New Song**  
```cql
INSERT INTO songs (song_id, title, artist, album, duration, genre, release_year) 
VALUES (uuid(), 'Shape of You', 'Ed Sheeran', 'Divide', 233, 'Pop', 2017);
```

---

## **📌 2. Table: Storing User Playlists**  
🔹 _Tracks the songs that users have added to their playlists._  

```cql
CREATE TABLE playlists (
    user_id UUID,  
    song_id UUID,  
    added_at timestamp,  
    playlist_name text,  
    PRIMARY KEY ((user_id, playlist_name), added_at, song_id)
) WITH CLUSTERING ORDER BY (added_at DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `(user_id, playlist_name)` → Groups songs **by user and playlist**.  
✔ **Clustering Key:** `added_at DESC, song_id` → Ensures **recently added songs appear first**.  

### **📌 Query Examples:**  
✔ **Retrieve All Songs in a User's Playlist**  
```cql
SELECT * FROM playlists WHERE user_id = 1234 AND playlist_name = 'My Favorites';
```

✔ **Add a Song to a Playlist**  
```cql
INSERT INTO playlists (user_id, song_id, added_at, playlist_name) 
VALUES (1234, 5678, toTimestamp(now()), 'My Favorites');
```

✔ **Remove a Song from a Playlist**  
```cql
DELETE FROM playlists WHERE user_id = 1234 AND playlist_name = 'My Favorites' AND song_id = 5678;
```

---

## **📌 3. Table: Searching Songs by Genre and Artist**  
🔹 _Allows users to explore songs by category._  

```cql
CREATE TABLE songs_by_genre_artist (
    genre text,  
    artist text,  
    song_id UUID,  
    title text,  
    PRIMARY KEY ((genre, artist), song_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `(genre, artist)` → Groups songs **by genre and artist**.  
✔ **Clustering Key:** `song_id` → Ensures songs **can be retrieved efficiently**.  

### **📌 Query Examples:**  
✔ **Retrieve All Pop Songs by Ed Sheeran**  
```cql
SELECT * FROM songs_by_genre_artist WHERE genre = 'Pop' AND artist = 'Ed Sheeran';
```

✔ **Insert a Song into the Genre-Artist Index**  
```cql
INSERT INTO songs_by_genre_artist (genre, artist, song_id, title) 
VALUES ('Pop', 'Ed Sheeran', 5678, 'Shape of You');
```

---

## **📌 4. Table: Storing Recently Played Songs**  
🔹 _Tracks the last songs a user has listened to._  

```cql
CREATE TABLE recently_played (
    user_id UUID,  
    played_at timestamp,  
    song_id UUID,  
    PRIMARY KEY (user_id, played_at, song_id)
) WITH CLUSTERING ORDER BY (played_at DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `user_id` → Groups **recently played songs per user**.  
✔ **Clustering Key:** `played_at DESC, song_id` → Ensures **latest songs appear first**.  

### **📌 Query Examples:**  
✔ **Retrieve a User's Last 10 Played Songs**  
```cql
SELECT * FROM recently_played WHERE user_id = 1234 LIMIT 10;
```

✔ **Insert a New Played Song Record**  
```cql
INSERT INTO recently_played (user_id, played_at, song_id) 
VALUES (1234, toTimestamp(now()), 5678);
```

---

## **📌 5. Table: Storing User Preferences (For Recommendations)**  
🔹 _Stores user preferences for better recommendations._  

```cql
CREATE TABLE user_preferences (
    user_id UUID PRIMARY KEY,  
    favorite_genres set<text>,  
    favorite_artists set<text>
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `user_id` → Ensures **each user has a single preference entry**.  
✔ **Set Data Type:** Allows storing **multiple genres and artists efficiently**.  

### **📌 Query Examples:**  
✔ **Retrieve a User’s Favorite Genres and Artists**  
```cql
SELECT * FROM user_preferences WHERE user_id = 1234;
```

✔ **Update a User’s Preferences**  
```cql
UPDATE user_preferences 
SET favorite_genres = favorite_genres + {'Rock'}, 
    favorite_artists = favorite_artists + {'Coldplay'} 
WHERE user_id = 1234;
```

---

## **📌 Summary of Schema Design**
| **Table** | **Partition Key** | **Clustering Key** | **Use Case** |
|-----------|----------------|----------------|-------------|
| **songs** | `song_id` | None | Stores song metadata. |
| **playlists** | `(user_id, playlist_name)` | `added_at, song_id` | Tracks songs in user playlists. |
| **songs_by_genre_artist** | `(genre, artist)` | `song_id` | Allows searching songs by genre and artist. |
| **recently_played** | `user_id` | `played_at, song_id` | Stores the last played songs per user. |
| **user_preferences** | `user_id` | None | Stores favorite genres and artists per user. |

---

## **📌 Optimizations for Performance**
✔ **Use `LeveledCompactionStrategy (LCS)` for Playlists (Frequent Updates)**  
```cql
ALTER TABLE playlists 
WITH compaction = { 'class': 'LeveledCompactionStrategy' };
```
🔹 _Ensures **efficient updates when users modify playlists.**_

✔ **Enable Caching for Frequently Accessed Songs**  
```cql
ALTER TABLE songs WITH caching = { 'keys': 'ALL', 'rows_per_partition': '100' };
```
🔹 _Speeds up **retrieving song metadata.**_

✔ **Use TTL for Auto-Deleting Old Recently Played Songs (Retain for 30 Days)**  
```cql
INSERT INTO recently_played (user_id, played_at, song_id) 
VALUES (1234, toTimestamp(now()), 5678) USING TTL 2592000;
```
🔹 _Deletes **old listening history after 30 days.**_

---


<br/>
<br/>

# **2. Designing a Cassandra Schema for a Distributed Blog Platform🔥🔥**  

### **🔹 Key Requirements:**  
✔ **Efficiently retrieve a user’s latest blog posts in chronological order.**  
✔ **Support fast pagination of posts.**  
✔ **Allow querying posts within a specific time range.**  
✔ **Scale to millions of users and blog posts.**  

---

## **📌 1. Table: Storing User Blog Posts**  
🔹 _Each post is stored with a `timeuuid` so that posts are automatically sorted by creation time._  

```cql
CREATE TABLE posts (
    user_id UUID,  
    post_id timeuuid,  
    title text,  
    content text,  
    tags set<text>,  
    created_at timestamp,  
    PRIMARY KEY (user_id, post_id)
) WITH CLUSTERING ORDER BY (post_id DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `user_id` → Groups **all posts for a user** together.  
✔ **Clustering Key:** `post_id` → Orders posts **by creation time (latest first)**.  
✔ **`timeuuid` for `post_id`** → Ensures **chronological sorting** of posts.  
✔ **Set Data Type (`tags`)** → Allows storing **multiple tags** per post.  

### **📌 Query Examples:**  
✔ **Retrieve the Latest 10 Posts by a User**  
```cql
SELECT * FROM posts WHERE user_id = 1234 LIMIT 10;
```

✔ **Retrieve Posts Within a Specific Time Range**  
```cql
SELECT * FROM posts WHERE user_id = 1234 
AND post_id >= minTimeuuid('2024-07-10 00:00:00') 
AND post_id <= maxTimeuuid('2024-07-10 23:59:59');
```

✔ **Insert a New Blog Post**  
```cql
INSERT INTO posts (user_id, post_id, title, content, tags, created_at) 
VALUES (1234, now(), 'My First Blog', 'This is my first blog post!', {'Tech', 'AI'}, toTimestamp(now()));
```

✔ **Delete a Blog Post**  
```cql
DELETE FROM posts WHERE user_id = 1234 AND post_id = 5678;
```

---

## **📌 2. Table: Searching Blog Posts by Tags**  
🔹 _Allows discovering blog posts based on categories._  

```cql
CREATE TABLE posts_by_tag (
    tag text,  
    post_id timeuuid,  
    user_id UUID,  
    title text,  
    PRIMARY KEY (tag, post_id)
) WITH CLUSTERING ORDER BY (post_id DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `tag` → Groups posts **by category or topic**.  
✔ **Clustering Key:** `post_id DESC` → Ensures **latest posts appear first**.  

### **📌 Query Examples:**  
✔ **Retrieve the Latest Posts for a Tag (e.g., "Tech")**  
```cql
SELECT * FROM posts_by_tag WHERE tag = 'Tech' LIMIT 10;
```

✔ **Insert a Post into the Tag-Based Index**  
```cql
INSERT INTO posts_by_tag (tag, post_id, user_id, title) 
VALUES ('Tech', now(), 1234, 'My First Blog');
```

---

## **📌 3. Table: Tracking User Comments on Blog Posts**  
🔹 _Stores comments on a post, sorted by timestamp._  

```cql
CREATE TABLE post_comments (
    post_id timeuuid,  
    comment_id timeuuid,  
    user_id UUID,  
    comment_text text,  
    created_at timestamp,  
    PRIMARY KEY (post_id, comment_id)
) WITH CLUSTERING ORDER BY (comment_id DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `post_id` → Groups **all comments for a post** together.  
✔ **Clustering Key:** `comment_id DESC` → Ensures **latest comments appear first**.  

### **📌 Query Examples:**  
✔ **Retrieve the Latest 5 Comments on a Post**  
```cql
SELECT * FROM post_comments WHERE post_id = 5678 LIMIT 5;
```

✔ **Insert a New Comment**  
```cql
INSERT INTO post_comments (post_id, comment_id, user_id, comment_text, created_at) 
VALUES (5678, now(), 4321, 'Great post!', toTimestamp(now()));
```

---

## **📌 4. Table: Storing User Likes on Posts**  
🔹 _Tracks which users have liked which blog posts._  

```cql
CREATE TABLE post_likes (
    post_id timeuuid,  
    user_id UUID,  
    liked_at timestamp,  
    PRIMARY KEY (post_id, user_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `post_id` → Groups **likes per post**.  
✔ **Clustering Key:** `user_id` → Ensures **efficient lookups of who liked the post**.  

### **📌 Query Examples:**  
✔ **Retrieve All Users Who Liked a Post**  
```cql
SELECT user_id FROM post_likes WHERE post_id = 5678;
```

✔ **Insert a New Like for a Post**  
```cql
INSERT INTO post_likes (post_id, user_id, liked_at) 
VALUES (5678, 4321, toTimestamp(now()));
```

---

## **📌 5. Table: Storing User Subscriptions (For Personalized Feeds)**  
🔹 _Tracks which users follow which other users._  

```cql
CREATE TABLE user_followers (
    user_id UUID,  
    follower_id UUID,  
    followed_at timestamp,  
    PRIMARY KEY (user_id, follower_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `user_id` → Stores **who follows a user**.  
✔ **Clustering Key:** `follower_id` → Ensures **uniqueness per follower**.  

### **📌 Query Examples:**  
✔ **Retrieve a User’s Followers**  
```cql
SELECT follower_id FROM user_followers WHERE user_id = 1234;
```

✔ **Follow a User**  
```cql
INSERT INTO user_followers (user_id, follower_id, followed_at) 
VALUES (1234, 5678, toTimestamp(now()));
```

✔ **Unfollow a User**  
```cql
DELETE FROM user_followers WHERE user_id = 1234 AND follower_id = 5678;
```

---

## **📌 Summary of Schema Design**
| **Table** | **Partition Key** | **Clustering Key** | **Use Case** |
|-----------|----------------|----------------|-------------|
| **posts** | `user_id` | `post_id` | Stores user blog posts. |
| **posts_by_tag** | `tag` | `post_id` | Enables tag-based post discovery. |
| **post_comments** | `post_id` | `comment_id` | Stores user comments on posts. |
| **post_likes** | `post_id` | `user_id` | Tracks likes per post. |
| **user_followers** | `user_id` | `follower_id` | Manages user subscriptions. |

---

## **📌 Optimizations for Performance**
✔ **Use `TimeWindowCompactionStrategy (TWCS)` for Efficient Post Storage**  
```cql
ALTER TABLE posts 
WITH compaction = { 'class': 'TimeWindowCompactionStrategy', 'compaction_window_size': '1', 'compaction_window_unit': 'DAYS' };
```
🔹 _Groups posts into **daily SSTables** for optimized retrieval._

✔ **Enable Caching for Frequently Accessed Blog Posts**  
```cql
ALTER TABLE posts WITH caching = { 'keys': 'ALL', 'rows_per_partition': '100' };
```
🔹 _Speeds up retrieving **popular posts**._

✔ **Use TTL for Auto-Deleting Old Comments (Retain for 1 Year)**  
```cql
INSERT INTO post_comments (post_id, comment_id, user_id, comment_text, created_at) 
VALUES (5678, now(), 4321, 'Nice blog!', toTimestamp(now())) USING TTL 31536000;
```
🔹 _Deletes **old comments automatically after 1 year.**_

---

## **📌 Next Steps**
Would you like me to:  
✔ **Add a real-time trending posts feature?**  
✔ **Provide real-world CQL scripts for blog platform testing?**  
✔ **Suggest best practices for integrating Cassandra with a recommendation engine?**  

Let me know how you'd like to proceed! 🚀📝

<br/>
<br/>

# **3. Designing a Real-Time Chat Application Using Cassandra🔥🔥**  

### **🔹 Key Requirements:**  
✔ **Efficiently store and retrieve messages in chronological order.**  
✔ **Ensure all messages are delivered to all participants.**  
✔ **Support quick retrieval of older messages (chat history).**  
✔ **Scale to handle millions of active users and messages.**  

---

## **📌 1. Table: Storing Chat Messages**  
🔹 _Stores messages per chat, ensuring retrieval in chronological order._  

```cql
CREATE TABLE messages (
    chat_id UUID,  
    message_time timestamp,  
    user_id UUID,  
    message text,  
    PRIMARY KEY (chat_id, message_time)
) WITH CLUSTERING ORDER BY (message_time ASC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `chat_id` → Groups **all messages for a conversation** together.  
✔ **Clustering Key:** `message_time ASC` → Ensures **messages are stored chronologically**.  
✔ **Retrieves chat history efficiently using range queries.**  

### **📌 Query Examples:**  
✔ **Retrieve the Last 50 Messages for a Chat**  
```cql
SELECT * FROM messages WHERE chat_id = 5678 LIMIT 50;
```

✔ **Retrieve Messages from a Specific Time Range**  
```cql
SELECT * FROM messages WHERE chat_id = 5678 
AND message_time >= '2024-07-10 12:00:00' 
AND message_time <= '2024-07-10 14:00:00';
```

✔ **Insert a New Chat Message**  
```cql
INSERT INTO messages (chat_id, message_time, user_id, message) 
VALUES (5678, toTimestamp(now()), 1234, 'Hello, how are you?');
```

✔ **Delete a Message (Soft Delete Approach Recommended)**  
```cql
DELETE FROM messages WHERE chat_id = 5678 AND message_time = '2024-07-10 12:05:00';
```

---

## **📌 2. Table: Tracking Message Delivery Status**  
🔹 _Ensures messages are delivered to all participants._  

```cql
CREATE TABLE message_delivery (
    message_id UUID,  
    user_id UUID,  
    delivered boolean,  
    read boolean,  
    delivered_at timestamp,  
    read_at timestamp,  
    PRIMARY KEY (message_id, user_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `message_id` → Tracks **who received each message**.  
✔ **Clustering Key:** `user_id` → Ensures **uniqueness per message-user pair**.  
✔ **Tracks whether a message was delivered and read.**  

### **📌 Query Examples:**  
✔ **Retrieve All Users Who Have Not Received a Message Yet**  
```cql
SELECT user_id FROM message_delivery WHERE message_id = 9876 AND delivered = false;
```

✔ **Mark a Message as Delivered to a User**  
```cql
UPDATE message_delivery 
SET delivered = true, delivered_at = toTimestamp(now()) 
WHERE message_id = 9876 AND user_id = 1234;
```

✔ **Mark a Message as Read**  
```cql
UPDATE message_delivery 
SET read = true, read_at = toTimestamp(now()) 
WHERE message_id = 9876 AND user_id = 1234;
```

---

## **📌 3. Table: Storing User Conversations (For Quick Access to Recent Chats)**  
🔹 _Tracks the last message for each conversation._  

```cql
CREATE TABLE user_chats (
    user_id UUID,  
    chat_id UUID,  
    last_message_time timestamp,  
    last_message text,  
    unread_count int,  
    PRIMARY KEY (user_id, chat_id)
) WITH CLUSTERING ORDER BY (last_message_time DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `user_id` → Stores **conversations per user**.  
✔ **Clustering Key:** `chat_id` → Allows **quick lookups of recent chats**.  
✔ **Unread Count Column:** Tracks **how many messages a user hasn’t read**.  

### **📌 Query Examples:**  
✔ **Retrieve All Chats for a User (Most Recent First)**  
```cql
SELECT * FROM user_chats WHERE user_id = 1234;
```

✔ **Insert or Update the Last Message in a Chat**  
```cql
INSERT INTO user_chats (user_id, chat_id, last_message_time, last_message, unread_count) 
VALUES (1234, 5678, toTimestamp(now()), 'Hey, are you there?', 1);
```

✔ **Update the Unread Count When User Reads Messages**  
```cql
UPDATE user_chats SET unread_count = 0 WHERE user_id = 1234 AND chat_id = 5678;
```

---

## **📌 4. Table: Storing Group Chat Participants**  
🔹 _Tracks who is in each chat group._  

```cql
CREATE TABLE chat_participants (
    chat_id UUID,  
    user_id UUID,  
    added_at timestamp,  
    PRIMARY KEY (chat_id, user_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `chat_id` → Groups **all users in a conversation**.  
✔ **Clustering Key:** `user_id` → Allows **quick lookup of participants**.  

### **📌 Query Examples:**  
✔ **Retrieve All Users in a Chat**  
```cql
SELECT user_id FROM chat_participants WHERE chat_id = 5678;
```

✔ **Insert a User into a Group Chat**  
```cql
INSERT INTO chat_participants (chat_id, user_id, added_at) 
VALUES (5678, 4321, toTimestamp(now()));
```

✔ **Remove a User from a Chat**  
```cql
DELETE FROM chat_participants WHERE chat_id = 5678 AND user_id = 4321;
```

---

## **📌 Summary of Schema Design**
| **Table** | **Partition Key** | **Clustering Key** | **Use Case** |
|-----------|----------------|----------------|-------------|
| **messages** | `chat_id` | `message_time` | Stores all messages per conversation. |
| **message_delivery** | `message_id` | `user_id` | Tracks message delivery and read status. |
| **user_chats** | `user_id` | `chat_id` | Tracks recent chats and unread messages. |
| **chat_participants** | `chat_id` | `user_id` | Stores participants for group chats. |

---

## **📌 Optimizations for Performance**
✔ **Use `TimeWindowCompactionStrategy (TWCS)` for Efficient Message Storage**  
```cql
ALTER TABLE messages 
WITH compaction = { 'class': 'TimeWindowCompactionStrategy', 'compaction_window_size': '1', 'compaction_window_unit': 'HOURS' };
```
🔹 _Groups messages into **hourly SSTables** for optimized retrieval._

✔ **Enable Caching for Frequently Accessed Conversations**  
```cql
ALTER TABLE user_chats WITH caching = { 'keys': 'ALL', 'rows_per_partition': '100' };
```
🔹 _Speeds up retrieving **recent chats**._

✔ **Use TTL for Auto-Deleting Old Messages (Retain for 6 Months)**  
```cql
INSERT INTO messages (chat_id, message_time, user_id, message) 
VALUES (5678, toTimestamp(now()), 1234, 'Hello!') USING TTL 15552000;
```
🔹 _Deletes **old messages automatically after 6 months.**_

---

## **📌 Next Steps**
Would you like me to:  
✔ **Add end-to-end encryption support for secure messaging?**  
✔ **Provide real-world CQL scripts for chat application testing?**  
✔ **Suggest best practices for integrating Cassandra with real-time messaging services (e.g., Kafka, WebSockets)?**  

Let me know how you'd like to proceed! 🚀💬

<br/>
<br/>

# **4. Designing a Cassandra Data Model for a Ride-Sharing Application (Uber/Lyft)🔥🔥**  

### **🔹 Key Requirements:**  
✔ **Efficiently store and track real-time locations of drivers and riders.**  
✔ **Allow quick retrieval of the latest location of a user.**  
✔ **Enable geospatial queries to find nearby drivers for ride matching.**  
✔ **Scale to millions of active users.**  

---

## **📌 1. Table: Storing Real-Time User Locations**  
🔹 _Stores the latest location data for each user (driver/rider)._  

```cql
CREATE TABLE locations (
    user_id UUID,  
    location_time timestamp,  
    lat decimal,  
    long decimal,  
    PRIMARY KEY (user_id, location_time)
) WITH CLUSTERING ORDER BY (location_time DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `user_id` → Groups **all location updates per user**.  
✔ **Clustering Key:** `location_time DESC` → Ensures **latest location appears first**.  
✔ **Allows querying historical location data for analytics.**  

### **📌 Query Examples:**  
✔ **Retrieve the Latest Location of a User**  
```cql
SELECT * FROM locations WHERE user_id = 1234 LIMIT 1;
```

✔ **Retrieve a User’s Location History for the Last Hour**  
```cql
SELECT * FROM locations WHERE user_id = 1234 
AND location_time >= '2024-07-10 12:00:00' 
AND location_time <= '2024-07-10 13:00:00';
```

✔ **Insert a New Location Update**  
```cql
INSERT INTO locations (user_id, location_time, lat, long) 
VALUES (1234, toTimestamp(now()), 37.7749, -122.4194);
```

✔ **Delete Old Location Data (Retain for 7 Days Only)**  
```cql
INSERT INTO locations (user_id, location_time, lat, long) 
VALUES (1234, toTimestamp(now()), 37.7749, -122.4194) USING TTL 604800;
```
🔹 _Deletes **location data automatically after 7 days (604,800 seconds).**_

---

## **📌 2. Table: Storing Geohashed Locations (For Proximity-Based Queries)**  
🔹 _Optimized for finding nearby drivers using geohashing._  

```cql
CREATE TABLE driver_locations (
    geohash text,  -- Encodes latitude & longitude into a compact string
    driver_id UUID,  
    location_time timestamp,  
    lat decimal,  
    long decimal,  
    status text,  -- "AVAILABLE", "ON_TRIP"
    PRIMARY KEY ((geohash), driver_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `geohash` → Groups **drivers by their approximate location**.  
✔ **Clustering Key:** `driver_id` → Ensures **uniqueness per driver**.  
✔ **Status Column:** Differentiates **available drivers vs. on-trip drivers.**  

### **📌 Query Examples:**  
✔ **Retrieve Available Drivers in a Specific Geohash Area**  
```cql
SELECT * FROM driver_locations WHERE geohash = '9q8yy' AND status = 'AVAILABLE';
```

✔ **Insert a New Driver Location Update**  
```cql
INSERT INTO driver_locations (geohash, driver_id, location_time, lat, long, status) 
VALUES ('9q8yy', 5678, toTimestamp(now()), 37.7749, -122.4194, 'AVAILABLE');
```

✔ **Update Driver Status When They Accept a Ride**  
```cql
UPDATE driver_locations SET status = 'ON_TRIP' 
WHERE geohash = '9q8yy' AND driver_id = 5678;
```

---

## **📌 3. Table: Storing Active Rides**  
🔹 _Tracks ongoing rides and their status._  

```cql
CREATE TABLE active_rides (
    ride_id UUID,  
    rider_id UUID,  
    driver_id UUID,  
    start_time timestamp,  
    end_time timestamp,  
    pickup_lat decimal,  
    pickup_long decimal,  
    dropoff_lat decimal,  
    dropoff_long decimal,  
    status text,  -- "REQUESTED", "ONGOING", "COMPLETED"
    PRIMARY KEY (ride_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `ride_id` → Ensures **uniqueness of each ride**.  
✔ **Status Column:** Tracks **real-time ride progress**.  

### **📌 Query Examples:**  
✔ **Retrieve an Active Ride by Ride ID**  
```cql
SELECT * FROM active_rides WHERE ride_id = 7890;
```

✔ **Insert a New Ride Request**  
```cql
INSERT INTO active_rides (ride_id, rider_id, driver_id, start_time, pickup_lat, pickup_long, status) 
VALUES (7890, 1234, 5678, toTimestamp(now()), 37.7749, -122.4194, 'REQUESTED');
```

✔ **Update Ride Status When Trip Ends**  
```cql
UPDATE active_rides SET end_time = toTimestamp(now()), status = 'COMPLETED' 
WHERE ride_id = 7890;
```

---

## **📌 4. Table: Storing Ride History (For Past Rides & Analytics)**  
🔹 _Stores completed rides separately for analytics and user history._  

```cql
CREATE TABLE ride_history (
    rider_id UUID,  
    ride_id UUID,  
    driver_id UUID,  
    start_time timestamp,  
    end_time timestamp,  
    pickup_lat decimal,  
    pickup_long decimal,  
    dropoff_lat decimal,  
    dropoff_long decimal,  
    fare decimal,  
    PRIMARY KEY (rider_id, ride_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `rider_id` → Groups rides **by user**.  
✔ **Clustering Key:** `ride_id` → Ensures **uniqueness per ride**.  

### **📌 Query Examples:**  
✔ **Retrieve a User’s Last 5 Rides**  
```cql
SELECT * FROM ride_history WHERE rider_id = 1234 LIMIT 5;
```

✔ **Insert a Completed Ride Record**  
```cql
INSERT INTO ride_history (rider_id, ride_id, driver_id, start_time, end_time, pickup_lat, pickup_long, dropoff_lat, dropoff_long, fare) 
VALUES (1234, 7890, 5678, '2024-07-10 12:00:00', '2024-07-10 12:30:00', 37.7749, -122.4194, 37.7854, -122.4010, 25.50);
```

---

## **📌 Summary of Schema Design**
| **Table** | **Partition Key** | **Clustering Key** | **Use Case** |
|-----------|----------------|----------------|-------------|
| **locations** | `user_id` | `location_time` | Stores real-time location updates. |
| **driver_locations** | `geohash` | `driver_id` | Enables geospatial queries for nearby drivers. |
| **active_rides** | `ride_id` | None | Tracks ongoing rides. |
| **ride_history** | `rider_id` | `ride_id` | Stores completed rides for analytics. |

---

## **📌 Optimizations for Performance**
✔ **Use `TimeWindowCompactionStrategy (TWCS)` for Efficient Location Storage**  
```cql
ALTER TABLE locations 
WITH compaction = { 'class': 'TimeWindowCompactionStrategy', 'compaction_window_size': '1', 'compaction_window_unit': 'HOURS' };
```
🔹 _Groups location updates into **hourly SSTables** for optimized retrieval._

✔ **Enable Caching for Frequently Accessed Ride Data**  
```cql
ALTER TABLE active_rides WITH caching = { 'keys': 'ALL', 'rows_per_partition': '100' };
```
🔹 _Speeds up **retrieving active rides**._

✔ **Use TTL for Auto-Deleting Old Location Data (Retain for 7 Days)**  
```cql
INSERT INTO locations (user_id, location_time, lat, long) 
VALUES (1234, toTimestamp(now()), 37.7749, -122.4194) USING TTL 604800;
```
🔹 _Deletes **old locations automatically after 7 days.**_

---

## **📌 Next Steps**
Would you like me to:  
✔ **Add surge pricing & fare calculation models?**  
✔ **Provide real-world CQL scripts for ride-sharing testing?**  
✔ **Suggest best practices for integrating Cassandra with real-time tracking systems (e.g., Kafka, Redis)?**  

Let me know how you'd like to proceed! 🚀🚖

<br/>
<br/>

# **5. Designing a Cassandra Data Model for a Large E-Commerce Website🔥🔥**  

### **🔹 Key Requirements:**  
✔ **List products by category efficiently.**  
✔ **Manage users' shopping carts for quick add/remove operations.**  
✔ **Store order history for retrieval and analytics.**  
✔ **Scale to handle millions of users and products.**  

---

## **📌 1. Table: Storing Product Information**  
🔹 _Allows searching for products by category._  

```cql
CREATE TABLE products (
    category_id int,  
    product_id UUID,  
    name text,  
    price decimal,  
    description text,  
    stock int,  
    PRIMARY KEY (category_id, product_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `category_id` → Groups **products by category**.  
✔ **Clustering Key:** `product_id` → Ensures **each product is uniquely identified**.  
✔ **Stock Column:** Tracks available **inventory for real-time updates**.  

### **📌 Query Examples:**  
✔ **Retrieve All Products in a Specific Category**  
```cql
SELECT * FROM products WHERE category_id = 101;
```

✔ **Retrieve a Specific Product by ID**  
```cql
SELECT * FROM products WHERE category_id = 101 AND product_id = 5678;
```

✔ **Insert a New Product**  
```cql
INSERT INTO products (category_id, product_id, name, price, description, stock) 
VALUES (101, uuid(), 'Wireless Headphones', 99.99, 'Noise-cancelling over-ear headphones', 50);
```

✔ **Update Stock After a Purchase**  
```cql
UPDATE products SET stock = stock - 1 WHERE category_id = 101 AND product_id = 5678;
```

---

## **📌 2. Table: Storing User Shopping Carts**  
🔹 _Tracks items users have added to their carts._  

```cql
CREATE TABLE carts (
    user_id UUID,  
    product_id UUID,  
    quantity int,  
    added_at timestamp,  
    PRIMARY KEY (user_id, product_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `user_id` → Groups **cart items per user**.  
✔ **Clustering Key:** `product_id` → Ensures **unique products per cart**.  

### **📌 Query Examples:**  
✔ **Retrieve All Items in a User's Cart**  
```cql
SELECT * FROM carts WHERE user_id = 1234;
```

✔ **Add a Product to a User's Cart**  
```cql
INSERT INTO carts (user_id, product_id, quantity, added_at) 
VALUES (1234, 5678, 2, toTimestamp(now()));
```

✔ **Update Quantity of an Item in the Cart**  
```cql
UPDATE carts SET quantity = 3 WHERE user_id = 1234 AND product_id = 5678;
```

✔ **Remove an Item from the Cart**  
```cql
DELETE FROM carts WHERE user_id = 1234 AND product_id = 5678;
```

✔ **Clear Cart After Checkout**  
```cql
DELETE FROM carts WHERE user_id = 1234;
```

---

## **📌 3. Table: Storing User Order History**  
🔹 _Tracks past purchases for analytics and user order history._  

```cql
CREATE TABLE orders (
    user_id UUID,  
    order_id timeuuid,  
    product_id UUID,  
    quantity int,  
    order_time timestamp,  
    total_price decimal,  
    status text,  -- "PENDING", "SHIPPED", "DELIVERED", "CANCELLED"
    PRIMARY KEY (user_id, order_id)
) WITH CLUSTERING ORDER BY (order_id DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `user_id` → Groups **orders per user**.  
✔ **Clustering Key:** `order_id DESC` → Ensures **latest orders appear first**.  
✔ **Status Column:** Tracks **order fulfillment stages**.  

### **📌 Query Examples:**  
✔ **Retrieve a User's Last 5 Orders**  
```cql
SELECT * FROM orders WHERE user_id = 1234 LIMIT 5;
```

✔ **Insert a New Order Record**  
```cql
INSERT INTO orders (user_id, order_id, product_id, quantity, order_time, total_price, status) 
VALUES (1234, now(), 5678, 2, toTimestamp(now()), 199.98, 'PENDING');
```

✔ **Update Order Status to Shipped**  
```cql
UPDATE orders SET status = 'SHIPPED' WHERE user_id = 1234 AND order_id = 9876;
```

✔ **Cancel an Order**  
```cql
UPDATE orders SET status = 'CANCELLED' WHERE user_id = 1234 AND order_id = 9876;
```

---

## **📌 4. Table: Tracking Product Reviews**  
🔹 _Allows users to leave feedback on purchased products._  

```cql
CREATE TABLE product_reviews (
    product_id UUID,  
    review_id timeuuid,  
    user_id UUID,  
    rating int,  -- Scale: 1 to 5
    review_text text,  
    created_at timestamp,  
    PRIMARY KEY (product_id, review_id)
) WITH CLUSTERING ORDER BY (review_id DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `product_id` → Groups **reviews per product**.  
✔ **Clustering Key:** `review_id DESC` → Ensures **latest reviews appear first**.  

### **📌 Query Examples:**  
✔ **Retrieve All Reviews for a Product**  
```cql
SELECT * FROM product_reviews WHERE product_id = 5678;
```

✔ **Insert a New Review**  
```cql
INSERT INTO product_reviews (product_id, review_id, user_id, rating, review_text, created_at) 
VALUES (5678, now(), 1234, 5, 'Great product!', toTimestamp(now()));
```

✔ **Retrieve Average Rating for a Product**  
```cql
SELECT avg(rating) FROM product_reviews WHERE product_id = 5678;
```

---

## **📌 Summary of Schema Design**
| **Table** | **Partition Key** | **Clustering Key** | **Use Case** |
|-----------|----------------|----------------|-------------|
| **products** | `category_id` | `product_id` | Stores products for quick retrieval by category. |
| **carts** | `user_id` | `product_id` | Tracks items in a user's shopping cart. |
| **orders** | `user_id` | `order_id` | Stores user order history. |
| **product_reviews** | `product_id` | `review_id` | Tracks product reviews and ratings. |

---

## **📌 Optimizations for Performance**
✔ **Use `LeveledCompactionStrategy (LCS)` for Shopping Carts (Frequent Updates)**  
```cql
ALTER TABLE carts 
WITH compaction = { 'class': 'LeveledCompactionStrategy' };
```
🔹 _Ensures efficient updates when users add/remove items from their carts._

✔ **Enable Caching for Frequently Accessed Product Data**  
```cql
ALTER TABLE products WITH caching = { 'keys': 'ALL', 'rows_per_partition': '100' };
```
🔹 _Speeds up retrieving **product details for browsing.**_

✔ **Use TTL for Auto-Deleting Old Shopping Cart Data (Retain for 30 Days)**  
```cql
INSERT INTO carts (user_id, product_id, quantity, added_at) 
VALUES (1234, 5678, 2, toTimestamp(now())) USING TTL 2592000;
```
🔹 _Deletes **abandoned cart items automatically after 30 days.**_

✔ **Use `TimeWindowCompactionStrategy (TWCS)` for Orders (Time-Series Data)**  
```cql
ALTER TABLE orders 
WITH compaction = { 'class': 'TimeWindowCompactionStrategy', 'compaction_window_size': '7', 'compaction_window_unit': 'DAYS' };
```
🔹 _Groups **weekly order records** into SSTables for optimized storage._

---

## **📌 Next Steps**
Would you like me to:  
✔ **Add support for real-time inventory tracking?**  
✔ **Provide real-world CQL scripts for testing order processing?**  
✔ **Suggest best practices for integrating Cassandra with a recommendation engine (e.g., collaborative filtering)?**  

Let me know how you'd like to proceed! 🚀🛒

<br/>
<br/>

# **6. Designing a Cassandra Data Model for Player Profiles in a Gaming Application🔥🔥**  

### **🔹 Key Requirements:**  
✔ **Frequent reads and occasional updates to player profiles.**  
✔ **Efficient retrieval of large profile data (achievements, stats, etc.).**  
✔ **Scalability to millions of players.**  
✔ **Optimize reads using caching strategies.**  

---

## **📌 1. Table: Storing Player Profiles**  
🔹 _Stores static player details and achievements._  

```cql
CREATE TABLE player_profiles (
    player_id UUID PRIMARY KEY,  
    username text,  
    email text,  
    achievements list<text>,  
    level int,  
    experience_points int,  
    avatar_url text,  
    last_login timestamp
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `player_id` → Ensures **each player has a unique profile.**  
✔ **List Data Type (`achievements`)** → Stores multiple **game achievements.**  
✔ **Optimized for infrequent updates but frequent reads.**  

### **📌 Query Examples:**  
✔ **Retrieve a Player's Profile**  
```cql
SELECT * FROM player_profiles WHERE player_id = 1234;
```

✔ **Insert a New Player Profile**  
```cql
INSERT INTO player_profiles (player_id, username, email, achievements, level, experience_points, avatar_url, last_login) 
VALUES (1234, 'Gamer123', 'gamer@example.com', ['First Kill', '100 Matches'], 5, 1500, 'http://example.com/avatar.jpg', toTimestamp(now()));
```

✔ **Update a Player’s Experience Points and Level**  
```cql
UPDATE player_profiles SET experience_points = 2000, level = 6 WHERE player_id = 1234;
```

✔ **Add a New Achievement to a Player Profile**  
```cql
UPDATE player_profiles SET achievements = achievements + ['Elite Sniper'] WHERE player_id = 1234;
```

---

## **📌 2. Table: Storing Game Progress Per Player**  
🔹 _Tracks player-specific game progress, such as levels completed, scores, etc._  

```cql
CREATE TABLE player_game_progress (
    player_id UUID,  
    game_id UUID,  
    last_level int,  
    high_score int,  
    last_played timestamp,  
    PRIMARY KEY (player_id, game_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `player_id` → Groups progress **by player.**  
✔ **Clustering Key:** `game_id` → Allows tracking **multiple games per player.**  
✔ **Efficient lookups for game progress.**  

### **📌 Query Examples:**  
✔ **Retrieve Player's Progress in a Game**  
```cql
SELECT * FROM player_game_progress WHERE player_id = 1234 AND game_id = 5678;
```

✔ **Insert or Update Player Progress**  
```cql
INSERT INTO player_game_progress (player_id, game_id, last_level, high_score, last_played) 
VALUES (1234, 5678, 12, 9800, toTimestamp(now()));
```

✔ **Update Player’s High Score in a Game**  
```cql
UPDATE player_game_progress SET high_score = 12000 WHERE player_id = 1234 AND game_id = 5678;
```

---

## **📌 3. Table: Tracking Player Inventory (For Virtual Items, Skins, Weapons, etc.)**  
🔹 _Stores items a player owns._  

```cql
CREATE TABLE player_inventory (
    player_id UUID,  
    item_id UUID,  
    item_name text,  
    quantity int,  
    acquired_at timestamp,  
    PRIMARY KEY (player_id, item_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `player_id` → Groups **all items per player.**  
✔ **Clustering Key:** `item_id` → Ensures **uniqueness per item.**  

### **📌 Query Examples:**  
✔ **Retrieve a Player's Inventory**  
```cql
SELECT * FROM player_inventory WHERE player_id = 1234;
```

✔ **Insert a New Item into a Player’s Inventory**  
```cql
INSERT INTO player_inventory (player_id, item_id, item_name, quantity, acquired_at) 
VALUES (1234, 9876, 'Golden Sword', 1, toTimestamp(now()));
```

✔ **Update Quantity of an Item**  
```cql
UPDATE player_inventory SET quantity = 2 WHERE player_id = 1234 AND item_id = 9876;
```

✔ **Remove an Item from Inventory**  
```cql
DELETE FROM player_inventory WHERE player_id = 1234 AND item_id = 9876;
```

---

## **📌 4. Table: Storing Friends & Social Connections**  
🔹 _Tracks friendships between players._  

```cql
CREATE TABLE player_friends (
    player_id UUID,  
    friend_id UUID,  
    added_at timestamp,  
    PRIMARY KEY (player_id, friend_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `player_id` → Groups **all friends per player.**  
✔ **Clustering Key:** `friend_id` → Ensures **uniqueness per friendship.**  

### **📌 Query Examples:**  
✔ **Retrieve a Player’s Friends List**  
```cql
SELECT friend_id FROM player_friends WHERE player_id = 1234;
```

✔ **Insert a New Friend Connection**  
```cql
INSERT INTO player_friends (player_id, friend_id, added_at) 
VALUES (1234, 5678, toTimestamp(now()));
```

✔ **Remove a Friend**  
```cql
DELETE FROM player_friends WHERE player_id = 1234 AND friend_id = 5678;
```

---

## **📌 Summary of Schema Design**
| **Table** | **Partition Key** | **Clustering Key** | **Use Case** |
|-----------|----------------|----------------|-------------|
| **player_profiles** | `player_id` | None | Stores static player profile details. |
| **player_game_progress** | `player_id` | `game_id` | Tracks player progress in different games. |
| **player_inventory** | `player_id` | `item_id` | Stores virtual items owned by players. |
| **player_friends** | `player_id` | `friend_id` | Tracks friendships between players. |

---

## **📌 Optimizations for Performance**
✔ **Use `LeveledCompactionStrategy (LCS)` for Player Profiles (Rare Updates)**  
```cql
ALTER TABLE player_profiles 
WITH compaction = { 'class': 'LeveledCompactionStrategy' };
```
🔹 _Optimized for **frequent reads, infrequent updates.**_

✔ **Enable Caching for Frequently Accessed Player Profiles**  
```cql
ALTER TABLE player_profiles WITH caching = { 'keys': 'ALL', 'rows_per_partition': '100' };
```
🔹 _Improves **latency for frequent profile reads.**_

✔ **Use TTL for Auto-Deleting Expired Inventory Items (e.g., Temporary Boosts)**  
```cql
INSERT INTO player_inventory (player_id, item_id, item_name, quantity, acquired_at) 
VALUES (1234, 9876, 'Limited-Time XP Boost', 1, toTimestamp(now())) USING TTL 86400;
```
🔹 _Deletes **items automatically after 1 day (86,400 seconds).**_

✔ **Use `TimeWindowCompactionStrategy (TWCS)` for Game Progress (Time-Series Data)**  
```cql
ALTER TABLE player_game_progress 
WITH compaction = { 'class': 'TimeWindowCompactionStrategy', 'compaction_window_size': '7', 'compaction_window_unit': 'DAYS' };
```
🔹 _Groups **weekly game progress records** into SSTables for efficient retrieval._

---

## **📌 Next Steps**
Would you like me to:  
✔ **Add leaderboards for competitive ranking?**  
✔ **Provide real-world CQL scripts for simulating player actions?**  
✔ **Suggest best practices for integrating Cassandra with a game analytics engine (e.g., Apache Spark, Kafka)?**  



# **7. Modeling Social Media Feeds in Cassandra (Retrieving Friends' Posts)🔥**  

## **🔹 Problem Statement:**  
✔ **Efficiently retrieve posts from friends a user follows.**  
✔ **Optimize data retrieval for real-time feeds.**  
✔ **Scale to millions of users and posts.**  

---

## **📌 1. Table: Storing User Posts**  
🔹 _Each user’s posts are stored in a time-ordered manner for fast lookups._  

```cql
CREATE TABLE user_posts (
    user_id UUID,  
    post_id timeuuid,  
    content text,  
    created_at timestamp,  
    PRIMARY KEY (user_id, post_id)
) WITH CLUSTERING ORDER BY (post_id DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `user_id` → Groups all posts for a user.  
✔ **Clustering Key:** `post_id` → Ensures posts **are stored in descending order** for fast retrieval.  
✔ **`timeuuid` for `post_id`** → Guarantees **unique and sequential** ordering.  

### **📌 Query Examples:**  
✔ **Retrieve a User’s Last 10 Posts**  
```cql
SELECT * FROM user_posts WHERE user_id = 1234 LIMIT 10;
```

✔ **Insert a New Post**  
```cql
INSERT INTO user_posts (user_id, post_id, content, created_at) 
VALUES (1234, now(), 'Just had an amazing lunch!', toTimestamp(now()));
```

---

## **📌 2. Table: Storing User Friendships (Follow System)**  
🔹 _Tracks which users follow which friends._  

```cql
CREATE TABLE user_friends (
    user_id UUID,  
    friend_id UUID,  
    added_at timestamp,  
    PRIMARY KEY (user_id, friend_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `user_id` → Groups all followed friends per user.  
✔ **Clustering Key:** `friend_id` → Ensures unique follow relationships.  

### **📌 Query Examples:**  
✔ **Retrieve All Friends a User Follows**  
```cql
SELECT friend_id FROM user_friends WHERE user_id = 1234;
```

✔ **Insert a New Follow Relationship**  
```cql
INSERT INTO user_friends (user_id, friend_id, added_at) 
VALUES (1234, 5678, toTimestamp(now()));
```

✔ **Remove a Friend (Unfollow Action)**  
```cql
DELETE FROM user_friends WHERE user_id = 1234 AND friend_id = 5678;
```

---

## **📌 3. Optimized Feed Table: Storing Aggregated Friends’ Posts (Denormalization for Performance)**  
🔹 _Pre-aggregates posts from friends into a single feed table for faster access._  

```cql
CREATE TABLE user_feed (
    user_id UUID,  
    post_id timeuuid,  
    friend_id UUID,  
    content text,  
    created_at timestamp,  
    PRIMARY KEY (user_id, post_id)
) WITH CLUSTERING ORDER BY (post_id DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `user_id` → Each user has their own personalized feed.  
✔ **Clustering Key:** `post_id` → Orders posts in **reverse chronological order**.  
✔ **`friend_id` Column:** Identifies **which friend created the post**.  

### **📌 Query Examples:**  
✔ **Retrieve a User’s Feed (Latest 10 Posts from Friends)**  
```cql
SELECT * FROM user_feed WHERE user_id = 1234 LIMIT 10;
```

✔ **Insert a Friend’s Post into a User’s Feed** (Denormalized Data)  
```cql
INSERT INTO user_feed (user_id, post_id, friend_id, content, created_at) 
VALUES (1234, now(), 5678, 'Amazing trip to Bali!', toTimestamp(now()));
```

---

## **📌 4. Alternative: Using a Materialized View for Faster Feed Retrieval**  
🔹 _Automatically creates a precomputed feed for users._  

```cql
CREATE MATERIALIZED VIEW user_feed_view AS 
SELECT user_id, post_id, friend_id, content, created_at 
FROM user_feed 
WHERE user_id IS NOT NULL PRIMARY KEY (user_id, post_id);
```

✔ **This allows retrieving posts efficiently without complex joins.**  

---

## **📌 How the Feed Retrieval Works Efficiently?**  
1️⃣ **Fetch the List of Friends a User Follows**  
```cql
SELECT friend_id FROM user_friends WHERE user_id = 1234;
```
2️⃣ **Fetch Latest Posts from Each Friend (Concurrent Reads)**  
```cql
SELECT * FROM user_posts WHERE user_id = 5678 LIMIT 5;
SELECT * FROM user_posts WHERE user_id = 6789 LIMIT 5;
...
```
3️⃣ **Combine Results in Application Code**  
✔ Fetching **directly from `user_feed`** is more efficient than making multiple individual queries.

---

## **📌 Summary of Schema Design**
| **Table** | **Partition Key** | **Clustering Key** | **Use Case** |
|-----------|----------------|----------------|-------------|
| **user_posts** | `user_id` | `post_id` | Store posts per user. |
| **user_friends** | `user_id` | `friend_id` | Track followed friends. |
| **user_feed** | `user_id` | `post_id` | Store aggregated posts for a user’s feed. |

---

## **📌 Optimizations for Performance**
✔ **Enable Caching for Frequently Accessed Feeds**  
```cql
ALTER TABLE user_feed WITH caching = { 'keys': 'ALL', 'rows_per_partition': '100' };
```

✔ **Use TTL for Auto-Deleting Old Feed Entries (After 30 Days)**  
```cql
INSERT INTO user_feed (user_id, post_id, friend_id, content, created_at) 
VALUES (1234, now(), 5678, 'Great weather today!', toTimestamp(now())) USING TTL 2592000;
```
🔹 _Deletes old feed posts after **30 days**._

✔ **Use Write-Back Strategies for Live Feeds**  
- Instead of querying multiple user posts, **precompute feeds when a post is created.**  

---

<br/>
<br/>

# **8. Configuring Multi-Datacenter Replication in Cassandra 🔥**  

### **🔹 Problem Statement:**  
✔ **Ensure low-latency reads/writes across multiple geographic regions.**  
✔ **Provide redundancy and fault tolerance for disaster recovery.**  
✔ **Optimize data placement to minimize cross-region network traffic.**  

---

## **📌 Solution: Using `NetworkTopologyStrategy`**  
🔹 _The **NetworkTopologyStrategy** replication strategy enables us to define **replication factors per data center**._  
🔹 _Cassandra intelligently places **replicas across different racks** to prevent data loss due to rack failures._  
🔹 _It **routes read and write requests to the nearest datacenter** to minimize latency._  

---

## **📌 1. Creating a Multi-Datacenter Keyspace**  
```cql
CREATE KEYSPACE mykeyspace 
WITH REPLICATION = { 
    'class' : 'NetworkTopologyStrategy', 
    'dc1' : 3,  -- Three replicas in Data Center 1
    'dc2' : 2   -- Two replicas in Data Center 2
};
```

### **✅ Explanation:**  
✔ **Replication per Data Center:**  
   - **`dc1: 3`** → 3 replicas in **Data Center 1**.  
   - **`dc2: 2`** → 2 replicas in **Data Center 2**.  
✔ **Fault Tolerance:** Each replica is placed in **a different rack** to prevent failures.  
✔ **Performance:** Cassandra routes queries to the **closest replica** in the same region.  

---

## **📌 2. Configuring Read and Write Consistency for Low Latency**  

### **✔ Low-Latency Writes (Using `LOCAL_QUORUM`)**  
```cql
INSERT INTO mytable (id, name) VALUES (1, 'Cassandra') USING CONSISTENCY LOCAL_QUORUM;
```
🔹 _Ensures **writes succeed** as long as a quorum (majority) of nodes in the local datacenter acknowledge the write._  

### **✔ Low-Latency Reads (Using `LOCAL_QUORUM`)**  
```cql
SELECT * FROM mytable WHERE id = 1 USING CONSISTENCY LOCAL_QUORUM;
```
🔹 _Ensures reads happen within the **local datacenter**, avoiding cross-region network latency._  

---

## **📌 3. Ensuring Data Synchronization Across Datacenters**  
🔹 _Cassandra uses **hinted handoff and repair** to keep data consistent across datacenters._  

### **✔ Manually Running Repair for Cross-Datacenter Consistency**  
```sh
nodetool repair -pr
```
🔹 _Synchronizes missing data across datacenters._  

### **✔ Monitoring Cross-Datacenter Replication Latency**  
```sh
nodetool status
```
🔹 _Ensures all nodes across `dc1` and `dc2` are **healthy and in sync**._  

---

## **📌 4. Query Routing Based on Locality**  

### **✔ Configuring Cassandra to Route Queries Locally**  
Modify **cassandra.yaml** in each datacenter:  
```yaml
endpoint_snitch: GossipingPropertyFileSnitch
```
🔹 _The **GossipingPropertyFileSnitch** ensures **Cassandra routes requests to the closest datacenter**._  

### **✔ Setting Up Client-Side Query Routing**  
In the **Cassandra driver configuration**, set **DCAwareRoundRobinPolicy** to prioritize local queries:  
```java
Cluster cluster = Cluster.builder()
    .addContactPoint("10.0.0.1")  // Local datacenter node
    .withLoadBalancingPolicy(DCAwareRoundRobinPolicy.builder()
        .withLocalDc("dc1")  // Prioritize local DC
        .build())
    .build();
```
🔹 _Ensures queries are **first attempted in the local datacenter** before querying remote ones._  

---

## **📌 5. Backup and Disaster Recovery Across Datacenters**  

### **✔ Taking a Snapshot Backup from a Datacenter**  
```sh
nodetool snapshot mykeyspace
```
🔹 _Creates a backup of all tables in `mykeyspace`._  

### **✔ Restoring Data to Another Datacenter (After Failure)**  
1️⃣ **Copy the backup from `dc1` to `dc2`**  
```sh
scp -r /var/lib/cassandra/data/mykeyspace/ user@dc2:/var/lib/cassandra/data/mykeyspace/
```
2️⃣ **Restart Cassandra on `dc2`**  
```sh
sudo systemctl restart cassandra
```

---

## **📌 Summary of Multi-Datacenter Replication Strategy**
| **Configuration** | **Purpose** |
|------------------|-------------|
| **NetworkTopologyStrategy** | Ensures **replication per datacenter** for redundancy. |
| **LOCAL_QUORUM Consistency** | Reduces **latency** by reading/writing in the local DC first. |
| **GossipingPropertyFileSnitch** | Routes requests to the **closest** datacenter. |
| **DCAwareRoundRobinPolicy** | Ensures **client-side load balancing** favors local nodes. |
| **nodetool repair** | Synchronizes **data across datacenters** periodically. |
| **Snapshot Backups** | Prevents **data loss in case of failures**. |

---

## **📌 Next Steps**
Would you like me to:  
✔ **Provide a real-world configuration for a production environment?**  
✔ **Optimize consistency settings for a specific workload?**  
✔ **Suggest best practices for handling failovers between datacenters?**  

<br/>
<br/>

# **9. Designing a Cassandra Data Model for a Stock Market Application🔥**  

### **🔹 Key Requirements:**  
✔ **Store and retrieve real-time stock prices efficiently.**  
✔ **Support fast querying of historical stock data.**  
✔ **Handle massive write throughput as stock prices update frequently.**  
✔ **Enable range queries to analyze stock trends.**  

---

## **📌 1. Table: Storing Real-Time and Historical Stock Prices**  
🔹 _Uses a **time-series wide-row model**, storing stock prices by symbol and timestamp._  

```cql
CREATE TABLE stock_prices (
    stock_symbol text,  
    price_time timestamp,  
    price decimal,  
    volume bigint,  -- Number of shares traded at this time
    PRIMARY KEY (stock_symbol, price_time)
) WITH CLUSTERING ORDER BY (price_time DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `stock_symbol` → Groups prices **by stock** for quick lookups.  
✔ **Clustering Key:** `price_time DESC` → Ensures newest prices **appear first**.  
✔ **Volume Tracking:** Helps in **technical analysis** of trading activity.  

### **📌 Query Examples:**  
✔ **Retrieve the Latest Price for a Stock**  
```cql
SELECT * FROM stock_prices WHERE stock_symbol = 'AAPL' LIMIT 1;
```

✔ **Retrieve All Prices for a Stock in the Last 7 Days**  
```cql
SELECT * FROM stock_prices WHERE stock_symbol = 'AAPL' 
AND price_time >= '2024-07-01' AND price_time <= '2024-07-08';
```

✔ **Insert a New Price Update**  
```cql
INSERT INTO stock_prices (stock_symbol, price_time, price, volume) 
VALUES ('AAPL', toTimestamp(now()), 189.50, 500000);
```

✔ **Auto-Delete Old Prices (Retain Only 1 Year of Data)**  
```cql
INSERT INTO stock_prices (stock_symbol, price_time, price, volume) 
VALUES ('AAPL', toTimestamp(now()), 189.50, 500000) USING TTL 31536000;
```
🔹 _Deletes price records **after 1 year (31536000 seconds).**_

---

## **📌 2. Table: Storing Daily Stock Summary (OHLC - Open, High, Low, Close Prices)**  
🔹 _Stores aggregated daily stock data for trend analysis._  

```cql
CREATE TABLE daily_stock_summary (
    stock_symbol text,  
    trade_date date,  
    open_price decimal,  
    high_price decimal,  
    low_price decimal,  
    close_price decimal,  
    volume bigint,  
    PRIMARY KEY (stock_symbol, trade_date)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `stock_symbol` → Stores data per stock.  
✔ **Clustering Key:** `trade_date` → Ensures retrieval **in chronological order**.  
✔ **OHLC Prices:** Essential for **candlestick charting & stock analysis.**  

### **📌 Query Examples:**  
✔ **Retrieve a Stock’s Daily Summary for the Last Month**  
```cql
SELECT * FROM daily_stock_summary WHERE stock_symbol = 'AAPL' 
AND trade_date >= '2024-06-01' AND trade_date <= '2024-06-30';
```

✔ **Insert Daily Summary for a Stock**  
```cql
INSERT INTO daily_stock_summary (stock_symbol, trade_date, open_price, high_price, low_price, close_price, volume) 
VALUES ('AAPL', '2024-07-10', 188.00, 190.50, 187.20, 189.30, 1000000);
```

---

## **📌 3. Table: Tracking Real-Time Market Trends**  
🔹 _Stores real-time price movements to track stock fluctuations within a trading day._  

```cql
CREATE TABLE market_trends (
    stock_symbol text,  
    trend_time timestamp,  
    trend_type text,  -- "UP", "DOWN", "STABLE"
    percent_change decimal,  
    PRIMARY KEY (stock_symbol, trend_time)
) WITH CLUSTERING ORDER BY (trend_time DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `stock_symbol` → Allows tracking trends per stock.  
✔ **Clustering Key:** `trend_time DESC` → Enables **retrieving recent trends quickly**.  

### **📌 Query Examples:**  
✔ **Retrieve the Last 5 Trend Movements for a Stock**  
```cql
SELECT * FROM market_trends WHERE stock_symbol = 'AAPL' LIMIT 5;
```

✔ **Insert a New Market Trend Data**  
```cql
INSERT INTO market_trends (stock_symbol, trend_time, trend_type, percent_change) 
VALUES ('AAPL', toTimestamp(now()), 'UP', 2.5);
```

---

## **📌 4. Table: Storing Stock Trading Transactions**  
🔹 _Tracks buy/sell orders placed by users._  

```cql
CREATE TABLE stock_trades (
    trade_id UUID,  
    user_id UUID,  
    stock_symbol text,  
    trade_time timestamp,  
    trade_type text,  -- "BUY", "SELL"
    quantity int,  
    price decimal,  
    PRIMARY KEY (trade_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `trade_id` → Ensures **uniqueness of each trade**.  
✔ **Trade Type & Quantity:** Tracks **buy/sell transactions** efficiently.  

### **📌 Query Examples:**  
✔ **Retrieve All Trades for a User**  
```cql
SELECT * FROM stock_trades WHERE user_id = 5678;
```

✔ **Insert a New Trade Transaction**  
```cql
INSERT INTO stock_trades (trade_id, user_id, stock_symbol, trade_time, trade_type, quantity, price) 
VALUES (uuid(), 5678, 'AAPL', toTimestamp(now()), 'BUY', 50, 189.75);
```

---

## **📌 Summary of Schema Design**
| **Table** | **Partition Key** | **Clustering Key** | **Use Case** |
|-----------|----------------|----------------|-------------|
| **stock_prices** | `stock_symbol` | `price_time` | Stores real-time stock prices. |
| **daily_stock_summary** | `stock_symbol` | `trade_date` | Stores daily OHLC data. |
| **market_trends** | `stock_symbol` | `trend_time` | Tracks real-time stock trends. |
| **stock_trades** | `trade_id` | None | Tracks stock buy/sell transactions. |

---

## **📌 Optimizations for Performance**
✔ **Use `TimeWindowCompactionStrategy (TWCS)` for Time-Series Data**  
```cql
ALTER TABLE stock_prices 
WITH compaction = { 'class': 'TimeWindowCompactionStrategy', 'compaction_window_size': '1', 'compaction_window_unit': 'DAYS' };
```
🔹 _Optimizes performance by **storing daily price data in separate SSTables**._

✔ **Enable Caching for Frequently Accessed Data**  
```cql
ALTER TABLE daily_stock_summary WITH caching = { 'keys': 'ALL', 'rows_per_partition': '100' };
```
🔹 _Caches **stock summaries** for quick retrieval._  

✔ **Use TTL for Auto-Cleaning Old Data**  
```cql
INSERT INTO market_trends (stock_symbol, trend_time, trend_type, percent_change) 
VALUES ('AAPL', toTimestamp(now()), 'UP', 2.5) USING TTL 2592000;
```
🔹 _Deletes **old trend data after 30 days.**_

---

## **📌 Next Steps**
Would you like me to:  
✔ **Add real-time alerts for price changes?**  
✔ **Provide real-world CQL scripts for stock market simulations?**  
✔ **Suggest best practices for integrating Cassandra with a real-time analytics engine?**  

<br/>
<br/>

# **10. Designing a Cassandra Data Model for an IoT Application (Handling Sensor Data)🔥🔥**  

### **🔹 Key Requirements:**  
✔ **Efficiently store and process high-frequency sensor data.**  
✔ **Optimize for fast writes and periodic reads of recent data.**  
✔ **Ensure scalability to handle billions of sensor readings.**  
✔ **Support time-series queries for analytics and monitoring.**  

---

## **📌 1. Table: Storing Sensor Data (Time-Series Model)**  
🔹 _Each sensor’s readings are stored in time order for fast lookups._  

```cql
CREATE TABLE sensor_data (
    sensor_id UUID,  
    recorded_at timestamp,  
    value decimal,  
    unit text,  -- Optional (e.g., "Celsius", "Pa", "m/s")
    PRIMARY KEY (sensor_id, recorded_at)
) WITH CLUSTERING ORDER BY (recorded_at DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `sensor_id` → Groups all data for a single sensor.  
✔ **Clustering Key:** `recorded_at DESC` → Ensures **latest readings appear first**.  
✔ **Wide-Row Model:** Stores all readings for a sensor efficiently.  

### **📌 Query Examples:**  
✔ **Retrieve the Latest Reading for a Sensor**  
```cql
SELECT * FROM sensor_data WHERE sensor_id = 1234 LIMIT 1;
```

✔ **Retrieve All Readings for a Sensor in the Last Hour**  
```cql
SELECT * FROM sensor_data WHERE sensor_id = 1234 
AND recorded_at >= '2024-07-10 12:00:00' AND recorded_at <= '2024-07-10 13:00:00';
```

✔ **Insert a New Sensor Reading**  
```cql
INSERT INTO sensor_data (sensor_id, recorded_at, value, unit) 
VALUES (1234, toTimestamp(now()), 25.6, 'Celsius');
```

✔ **Auto-Delete Old Sensor Data (Retain Only 30 Days of Data)**  
```cql
INSERT INTO sensor_data (sensor_id, recorded_at, value, unit) 
VALUES (1234, toTimestamp(now()), 25.6, 'Celsius') USING TTL 2592000;
```
🔹 _Deletes readings **after 30 days (2,592,000 seconds)**._

---

## **📌 2. Choosing the Right Compaction Strategy**  

### **🔹 Best Compaction Strategies for IoT Data:**  

| **Strategy** | **Use Case** | **Pros** | **Cons** |
|-------------|-------------|---------|---------|
| **Time-Window Compaction Strategy (TWCS)** | **Best for time-series data** with frequent writes | Optimized for sequential writes, low read amplification | Slightly higher disk usage |
| **Date-Tiered Compaction Strategy (DTCS)** | Older alternative to TWCS, **good for large historical queries** | Reduces compaction overhead for older data | Can cause high read amplification |
| **Leveled Compaction Strategy (LCS)** | Best when **reads are more frequent than writes** | Ensures **low read latency**, good for real-time dashboards | Higher CPU & disk I/O usage |

### **✅ Recommended Strategy: `TimeWindowCompactionStrategy (TWCS)`**  
```cql
ALTER TABLE sensor_data 
WITH compaction = { 
    'class': 'TimeWindowCompactionStrategy', 
    'compaction_window_size': '1',  
    'compaction_window_unit': 'DAYS'
};
```
🔹 _Groups sensor data into **daily SSTables** for efficient storage._  
🔹 _Reduces read amplification for **recent data queries**._  

---

## **📌 3. Table: Aggregated Sensor Data (For Fast Retrieval of Historical Trends)**  
🔹 _Stores summarized data for efficient historical queries._  

```cql
CREATE TABLE sensor_daily_summary (
    sensor_id UUID,  
    date date,  
    avg_value decimal,  
    min_value decimal,  
    max_value decimal,  
    PRIMARY KEY (sensor_id, date)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `sensor_id` → Groups data by sensor.  
✔ **Clustering Key:** `date` → Enables **fast retrieval of daily trends**.  

### **📌 Query Examples:**  
✔ **Retrieve Last 7 Days of Sensor Data Summary**  
```cql
SELECT * FROM sensor_daily_summary WHERE sensor_id = 1234 
AND date >= '2024-07-04' AND date <= '2024-07-10';
```

✔ **Insert a Daily Summary Record**  
```cql
INSERT INTO sensor_daily_summary (sensor_id, date, avg_value, min_value, max_value) 
VALUES (1234, '2024-07-10', 25.5, 22.0, 28.3);
```

---

## **📌 4. Table: Real-Time Alerts for Anomalies**  
🔹 _Tracks when a sensor exceeds a threshold for anomaly detection._  

```cql
CREATE TABLE sensor_alerts (
    sensor_id UUID,  
    alert_time timestamp,  
    alert_type text,  
    value decimal,  
    message text,  
    PRIMARY KEY (sensor_id, alert_time)
) WITH CLUSTERING ORDER BY (alert_time DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `sensor_id` → Tracks alerts per sensor.  
✔ **Clustering Key:** `alert_time DESC` → Stores recent alerts **first**.  

### **📌 Query Examples:**  
✔ **Retrieve Last 5 Alerts for a Sensor**  
```cql
SELECT * FROM sensor_alerts WHERE sensor_id = 1234 LIMIT 5;
```

✔ **Insert a New Alert When a Sensor Exceeds a Threshold**  
```cql
INSERT INTO sensor_alerts (sensor_id, alert_time, alert_type, value, message) 
VALUES (1234, toTimestamp(now()), 'Temperature Spike', 85.3, 'Sensor exceeded safe threshold');
```

---

## **📌 Summary of Schema Design**
| **Table** | **Partition Key** | **Clustering Key** | **Use Case** |
|-----------|----------------|----------------|-------------|
| **sensor_data** | `sensor_id` | `recorded_at` | Stores real-time sensor readings. |
| **sensor_daily_summary** | `sensor_id` | `date` | Stores aggregated daily sensor data. |
| **sensor_alerts** | `sensor_id` | `alert_time` | Tracks real-time sensor anomalies. |

---

## **📌 Optimizations for Performance**
✔ **Enable TWCS for Efficient Time-Series Compaction**  
```cql
ALTER TABLE sensor_data 
WITH compaction = { 'class': 'TimeWindowCompactionStrategy', 'compaction_window_size': '1', 'compaction_window_unit': 'DAYS' };
```
🔹 _Ensures efficient storage and fast retrieval of recent data._

✔ **Use TTL to Auto-Expire Old Data**  
```cql
INSERT INTO sensor_data (sensor_id, recorded_at, value, unit) 
VALUES (1234, toTimestamp(now()), 26.3, 'Celsius') USING TTL 2592000;
```
🔹 _Deletes **older readings automatically after 30 days.**_

✔ **Use Caching for Frequently Accessed Data**  
```cql
ALTER TABLE sensor_daily_summary WITH caching = { 'keys': 'ALL', 'rows_per_partition': '100' };
```
🔹 _Speeds up **historical queries**._

---

## **📌 Next Steps**
Would you like me to:  
✔ **Add support for real-time dashboards using Apache Kafka?**  
✔ **Provide CQL scripts for simulating high-frequency sensor data ingestion?**  
✔ **Suggest best practices for integrating Cassandra with AI-based anomaly detection?**  

<br/>
<br/>

# **11. Designing a Cassandra Data Model for Tracking User Events and Timeline Display🔥🔥**  

### **🔹 Key Requirements:**  
✔ **Efficiently track user interactions (clicks, logins, purchases, etc.).**  
✔ **Support fast retrieval of user events for a timeline view.**  
✔ **Store and order events chronologically.**  
✔ **Scale to handle millions of users generating billions of events.**  

---

## **📌 1. Table: Storing User Events (Time-Ordered Model)**  
🔹 _Each user’s events are stored **chronologically** to support quick retrieval for timeline generation._  

```cql
CREATE TABLE user_events (
    user_id UUID,  
    event_time timestamp,  
    event_type text,  -- "LOGIN", "CLICK", "PURCHASE", etc.
    event_data text,  -- JSON/String with additional event details
    PRIMARY KEY (user_id, event_time)
) WITH CLUSTERING ORDER BY (event_time DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `user_id` → Groups events per user.  
✔ **Clustering Key:** `event_time DESC` → Ensures **latest events appear first**.  
✔ **Flexible Schema:** Stores event details in `event_data` (JSON-like format).  

### **📌 Query Examples:**  
✔ **Retrieve the Last 10 Events for a User**  
```cql
SELECT * FROM user_events WHERE user_id = 1234 LIMIT 10;
```

✔ **Retrieve Events Within a Specific Time Range**  
```cql
SELECT * FROM user_events WHERE user_id = 1234 
AND event_time >= '2024-07-10 12:00:00' AND event_time <= '2024-07-10 14:00:00';
```

✔ **Insert a New Event**  
```cql
INSERT INTO user_events (user_id, event_time, event_type, event_data) 
VALUES (1234, toTimestamp(now()), 'PURCHASE', '{"item_id": "9876", "price": "29.99"}');
```

✔ **Auto-Delete Old Events (Retain Only 90 Days of Data)**  
```cql
INSERT INTO user_events (user_id, event_time, event_type, event_data) 
VALUES (1234, toTimestamp(now()), 'LOGIN', '{}') USING TTL 7776000;
```
🔹 _Deletes events **after 90 days (7,776,000 seconds).**_

---

## **📌 2. Table: Aggregated User Activity (Daily Summary for Quick Lookups)**  
🔹 _Stores **daily** activity summaries to reduce query load on raw events._  

```cql
CREATE TABLE user_daily_activity (
    user_id UUID,  
    event_date date,  
    total_events int,  
    last_event_type text,  
    last_event_time timestamp,  
    PRIMARY KEY (user_id, event_date)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `user_id` → Groups activity per user.  
✔ **Clustering Key:** `event_date` → Stores daily summaries in chronological order.  
✔ **Precomputed Aggregations:** Reduces queries on `user_events` table.  

### **📌 Query Examples:**  
✔ **Retrieve a User’s Last 7 Days of Activity**  
```cql
SELECT * FROM user_daily_activity WHERE user_id = 1234 
AND event_date >= '2024-07-04' AND event_date <= '2024-07-10';
```

✔ **Insert a Daily Activity Summary Record**  
```cql
INSERT INTO user_daily_activity (user_id, event_date, total_events, last_event_type, last_event_time) 
VALUES (1234, '2024-07-10', 15, 'PURCHASE', '2024-07-10 18:45:00');
```

---

## **📌 3. Table: Global User Activity Feed (Public Events for Trending Analysis)**  
🔹 _Stores **public events** (e.g., post likes, shares, comments) for discovery feeds._  

```cql
CREATE TABLE global_events (
    event_time timestamp,  
    user_id UUID,  
    event_type text,  
    event_data text,  
    PRIMARY KEY (event_time, user_id)
) WITH CLUSTERING ORDER BY (user_id ASC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `event_time` → Groups all global events per time window.  
✔ **Clustering Key:** `user_id` → Supports filtering by user.  

### **📌 Query Examples:**  
✔ **Retrieve Last 50 Global Events (For News Feed, Trending, etc.)**  
```cql
SELECT * FROM global_events LIMIT 50;
```

✔ **Insert a New Global Event**  
```cql
INSERT INTO global_events (event_time, user_id, event_type, event_data) 
VALUES (toTimestamp(now()), 1234, 'POST_LIKE', '{"post_id": "5678"}');
```

---

## **📌 4. Alternative Approach: Using Materialized Views for Faster Retrieval**  
🔹 _Automatically creates **precomputed views** for user timelines._  

```cql
CREATE MATERIALIZED VIEW user_event_view AS 
SELECT user_id, event_time, event_type, event_data 
FROM user_events 
WHERE user_id IS NOT NULL PRIMARY KEY (event_time, user_id);
```

✔ **Reduces the need for complex range queries** on `user_events`._  

---

## **📌 Summary of Schema Design**
| **Table** | **Partition Key** | **Clustering Key** | **Use Case** |
|-----------|----------------|----------------|-------------|
| **user_events** | `user_id` | `event_time` | Stores all user events (e.g., logins, purchases). |
| **user_daily_activity** | `user_id` | `event_date` | Stores aggregated daily user activity. |
| **global_events** | `event_time` | `user_id` | Stores public user interactions for feeds. |

---

## **📌 Optimizations for Performance**
✔ **Use `TimeWindowCompactionStrategy (TWCS)` for Efficient Event Storage**  
```cql
ALTER TABLE user_events 
WITH compaction = { 'class': 'TimeWindowCompactionStrategy', 'compaction_window_size': '1', 'compaction_window_unit': 'DAYS' };
```
🔹 _Groups daily events into SSTables for **optimized time-series queries**._

✔ **Enable Caching for Frequently Accessed Data**  
```cql
ALTER TABLE user_daily_activity WITH caching = { 'keys': 'ALL', 'rows_per_partition': '100' };
```
🔹 _Improves retrieval speed for daily event summaries._

✔ **Use TTL to Auto-Expire Old Events**  
```cql
INSERT INTO global_events (event_time, user_id, event_type, event_data) 
VALUES (toTimestamp(now()), 1234, 'COMMENT', '{"post_id": "5678"}') USING TTL 2592000;
```
🔹 _Deletes **old global events automatically after 30 days.**_

---

## **📌 Next Steps**
Would you like me to:  
✔ **Add user notification tracking (e.g., unread event count)?**  
✔ **Provide real-world CQL scripts for timeline simulations?**  
✔ **Suggest best practices for integrating Cassandra with a real-time analytics engine (e.g., Apache Spark, Kafka)?**  

<br/>
<br/>


# **12. Designing a Distributed Task Queue with Priorities in Cassandra🔥🔥**  

### **🔹 Key Requirements:**  
✔ **Efficiently store and process tasks based on priority.**  
✔ **Ensure tasks are picked up and executed in the correct order.**  
✔ **Support different task statuses (`PENDING`, `IN_PROGRESS`, `COMPLETED`).**  
✔ **Distribute tasks across multiple nodes for scalability.**  

---

## **📌 1. Table: Storing Tasks in a Priority Queue**  
🔹 _Stores tasks categorized by status and priority._  

```cql
CREATE TABLE tasks (
    status text,  -- "PENDING", "IN_PROGRESS", "COMPLETED"
    priority int,  -- Lower value = higher priority (e.g., 1 is the highest priority)
    task_id UUID,  
    data text,  -- Task-specific information
    created_at timestamp,  
    PRIMARY KEY (status, priority, task_id)
) WITH CLUSTERING ORDER BY (priority ASC, task_id ASC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `status` → Groups tasks **by their execution state**.  
✔ **Clustering Key:** `priority ASC, task_id ASC` → Ensures **higher priority tasks come first**.  
✔ **Created Timestamp:** Allows tracking task creation time.  

### **📌 Query Examples:**  
✔ **Retrieve the Next Pending Task (Highest Priority First)**  
```cql
SELECT * FROM tasks WHERE status = 'PENDING' LIMIT 1;
```

✔ **Retrieve All Pending Tasks in Order of Priority**  
```cql
SELECT * FROM tasks WHERE status = 'PENDING';
```

✔ **Insert a New Task**  
```cql
INSERT INTO tasks (status, priority, task_id, data, created_at) 
VALUES ('PENDING', 1, uuid(), 'Process Order #5678', toTimestamp(now()));
```

✔ **Update a Task to In Progress**  
```cql
UPDATE tasks SET status = 'IN_PROGRESS' WHERE status = 'PENDING' AND priority = 1 AND task_id = 1234;
```

✔ **Mark a Task as Completed**  
```cql
UPDATE tasks SET status = 'COMPLETED' WHERE status = 'IN_PROGRESS' AND priority = 1 AND task_id = 1234;
```

---

## **📌 2. Handling Task Expiry (Auto-Delete Old Tasks)**  
🔹 _Use TTL to automatically remove old completed tasks._  

```cql
INSERT INTO tasks (status, priority, task_id, data, created_at) 
VALUES ('COMPLETED', 3, uuid(), 'Cleanup temp files', toTimestamp(now())) USING TTL 2592000;
```
🔹 _Deletes the task **after 30 days (2,592,000 seconds).**_

---

## **📌 3. Alternative Approach: Using Separate Tables for Task Processing**  

### **✔ Active Task Queue (Faster Retrieval of Pending Tasks)**
```cql
CREATE TABLE active_tasks (
    priority int,  
    task_id UUID,  
    data text,  
    created_at timestamp,  
    PRIMARY KEY (priority, task_id)
) WITH CLUSTERING ORDER BY (task_id ASC);
```
✔ _Stores only pending tasks for fast access._  
✔ _Once executed, tasks are **moved to `tasks_archive` instead of being updated**._

### **✔ Task Archive (Historical Record for Auditing)**
```cql
CREATE TABLE tasks_archive (
    task_id UUID PRIMARY KEY,  
    priority int,  
    status text,  
    data text,  
    completed_at timestamp
);
```
✔ _Stores completed/canceled tasks for **reporting & auditing.**_

---

## **📌 4. Optimizing Task Processing for Parallel Execution**  

### **✔ Using Materialized Views for Faster Retrieval**  
🔹 _Precomputes queries for different task statuses._  

```cql
CREATE MATERIALIZED VIEW pending_tasks AS 
SELECT task_id, priority, data FROM tasks 
WHERE status = 'PENDING' PRIMARY KEY (priority, task_id);
```
✔ _Allows fetching tasks efficiently without filtering._

---

## **📌 Summary of Schema Design**
| **Table** | **Partition Key** | **Clustering Key** | **Use Case** |
|-----------|----------------|----------------|-------------|
| **tasks** | `status` | `priority, task_id` | Stores all tasks categorized by status. |
| **active_tasks** | `priority` | `task_id` | Stores only pending tasks for faster retrieval. |
| **tasks_archive** | `task_id` | None | Stores completed and canceled tasks. |

---

## **📌 Optimizations for Performance**
✔ **Use `LeveledCompactionStrategy (LCS)` for Frequently Updated Data**  
```cql
ALTER TABLE tasks 
WITH compaction = { 'class': 'LeveledCompactionStrategy' };
```
🔹 _Ensures fast updates and prevents read amplification._

✔ **Enable Caching for Frequently Accessed Pending Tasks**  
```cql
ALTER TABLE active_tasks WITH caching = { 'keys': 'ALL', 'rows_per_partition': '100' };
```
🔹 _Speeds up **retrieval of pending tasks**._

✔ **Use TTL for Auto-Deleting Old Completed Tasks**  
```cql
INSERT INTO tasks_archive (task_id, priority, status, data, completed_at) 
VALUES (uuid(), 2, 'COMPLETED', 'Processed customer order', toTimestamp(now())) USING TTL 2592000;
```
🔹 _Removes **completed tasks automatically after 30 days.**_

---

## **📌 Next Steps**
Would you like me to:  
✔ **Add a mechanism for task retries (handling failed tasks)?**  
✔ **Provide real-world CQL scripts for distributed task queue simulation?**  
✔ **Suggest best practices for integrating Cassandra with message brokers (e.g., Kafka, RabbitMQ)?**  

<br/>
<br/>

# **📌13. Designing a Cassandra Schema for a Movie Recommendation System🔥🔥**  

### **🔹 Key Requirements:**  
✔ **Efficiently store and retrieve user ratings for movies.**  
✔ **Allow quick updates when a user changes their rating.**  
✔ **Support retrieving all ratings by a user and aggregating ratings for a movie.**  
✔ **Scale to millions of users and movies for personalized recommendations.**  

---

## **📌 1. Table: Storing User Ratings for Movies**  
🔹 _Each row represents a user’s rating for a movie._  

```cql
CREATE TABLE user_ratings (
    user_id UUID,  
    movie_id UUID,  
    rating int,  
    rated_at timestamp,  
    PRIMARY KEY (user_id, movie_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `user_id` → Groups all movies rated by a user.  
✔ **Clustering Key:** `movie_id` → Ensures efficient lookups by movie.  
✔ **Timestamp Column (`rated_at`)** → Tracks when the rating was given.  

### **📌 Query Examples:**  
✔ **Retrieve All Ratings by a User**  
```cql
SELECT * FROM user_ratings WHERE user_id = 1234;
```

✔ **Retrieve a User’s Rating for a Specific Movie**  
```cql
SELECT rating FROM user_ratings WHERE user_id = 1234 AND movie_id = 5678;
```

✔ **Insert or Update a Movie Rating**  
```cql
INSERT INTO user_ratings (user_id, movie_id, rating, rated_at) 
VALUES (1234, 5678, 5, toTimestamp(now()));
```

✔ **Update an Existing Rating**  
```cql
UPDATE user_ratings SET rating = 4 WHERE user_id = 1234 AND movie_id = 5678;
```

---

## **📌 2. Table: Aggregated Movie Ratings (For Fast Recommendation Queries)**  
🔹 _Stores average ratings and total votes for each movie._  

```cql
CREATE TABLE movie_ratings (
    movie_id UUID PRIMARY KEY,  
    total_ratings counter,  
    sum_ratings counter
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `movie_id` → Ensures **quick lookup per movie**.  
✔ **Counters for Aggregation:** `total_ratings` (number of ratings) & `sum_ratings` (sum of all ratings).  

### **📌 Query Examples:**  
✔ **Retrieve a Movie's Average Rating**  
```cql
SELECT sum_ratings, total_ratings FROM movie_ratings WHERE movie_id = 5678;
```
✔ **Insert a New Rating (Increment Rating Counters)**  
```cql
UPDATE movie_ratings SET total_ratings = total_ratings + 1, sum_ratings = sum_ratings + 5 WHERE movie_id = 5678;
```
✔ **Compute Average Rating in Application Code**  
```sql
AVG_RATING = sum_ratings / total_ratings
```

---

## **📌 3. Table: Storing Movies by Genre (For Discoverability)**  
🔹 _Allows users to find highly-rated movies by genre._  

```cql
CREATE TABLE movies_by_genre (
    genre text,  
    movie_id UUID,  
    title text,  
    avg_rating float,  
    PRIMARY KEY (genre, avg_rating, movie_id)
) WITH CLUSTERING ORDER BY (avg_rating DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `genre` → Groups all movies of the same genre.  
✔ **Clustering Key:** `avg_rating DESC, movie_id` → Ensures **highest-rated movies appear first**.  

### **📌 Query Examples:**  
✔ **Retrieve Top-Rated Movies in a Genre**  
```cql
SELECT * FROM movies_by_genre WHERE genre = 'Action' LIMIT 10;
```
✔ **Insert a Movie Record**  
```cql
INSERT INTO movies_by_genre (genre, movie_id, title, avg_rating) 
VALUES ('Action', 5678, 'Fast & Furious', 4.5);
```

---

## **📌 4. Table: Tracking User Movie Watch History**  
🔹 _Helps in providing personalized recommendations._  

```cql
CREATE TABLE user_watch_history (
    user_id UUID,  
    movie_id UUID,  
    watched_at timestamp,  
    PRIMARY KEY (user_id, watched_at, movie_id)
) WITH CLUSTERING ORDER BY (watched_at DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `user_id` → Groups watch history per user.  
✔ **Clustering Key:** `watched_at DESC, movie_id` → Stores **latest watched movies first**.  

### **📌 Query Examples:**  
✔ **Retrieve the Last 5 Movies Watched by a User**  
```cql
SELECT * FROM user_watch_history WHERE user_id = 1234 LIMIT 5;
```

✔ **Insert a Watched Movie Record**  
```cql
INSERT INTO user_watch_history (user_id, movie_id, watched_at) 
VALUES (1234, 5678, toTimestamp(now()));
```

---

## **📌 5. Table: Personalized Movie Recommendations (Precomputed Data for Fast Access)**  
🔹 _Stores precomputed recommendations for each user based on their history._  

```cql
CREATE TABLE user_recommendations (
    user_id UUID,  
    recommended_movie_id UUID,  
    reason text,  -- "Based on similar users" / "Based on your watch history"
    PRIMARY KEY (user_id, recommended_movie_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `user_id` → Stores recommendations per user.  
✔ **Clustering Key:** `recommended_movie_id` → Ensures quick access to recommended movies.  

### **📌 Query Examples:**  
✔ **Retrieve Movie Recommendations for a User**  
```cql
SELECT * FROM user_recommendations WHERE user_id = 1234;
```

✔ **Insert a Movie Recommendation for a User**  
```cql
INSERT INTO user_recommendations (user_id, recommended_movie_id, reason) 
VALUES (1234, 9876, 'Based on your watch history');
```

---

## **📌 Summary of Schema Design**
| **Table** | **Partition Key** | **Clustering Key** | **Use Case** |
|-----------|----------------|----------------|-------------|
| **user_ratings** | `user_id` | `movie_id` | Stores user ratings for movies. |
| **movie_ratings** | `movie_id` | None | Tracks total ratings and average rating for a movie. |
| **movies_by_genre** | `genre` | `avg_rating, movie_id` | Fetch top-rated movies by genre. |
| **user_watch_history** | `user_id` | `watched_at, movie_id` | Tracks user watch history for recommendations. |
| **user_recommendations** | `user_id` | `recommended_movie_id` | Precomputed recommendations for a user. |

---

## **📌 Optimizations for Performance**
✔ **Use `LeveledCompactionStrategy (LCS)` for Frequently Updated Ratings**  
```cql
ALTER TABLE user_ratings 
WITH compaction = { 'class': 'LeveledCompactionStrategy' };
```
🔹 _Ensures **efficient updates when users change ratings.**_

✔ **Enable Caching for Frequently Accessed Movie Ratings**  
```cql
ALTER TABLE movie_ratings WITH caching = { 'keys': 'ALL', 'rows_per_partition': '100' };
```
🔹 _Speeds up fetching **average ratings per movie.**_

✔ **Use TTL to Auto-Expire Old Watch History (Retain for 1 Year)**  
```cql
INSERT INTO user_watch_history (user_id, movie_id, watched_at) 
VALUES (1234, 5678, toTimestamp(now())) USING TTL 31536000;
```
🔹 _Deletes **old watch history automatically after 1 year.**_

---

## **📌 Next Steps**
Would you like me to:  
✔ **Add support for user reviews and comments on movies?**  
✔ **Provide real-world CQL scripts for running movie recommendations?**  
✔ **Suggest best practices for integrating Cassandra with AI-based recommendation engines (e.g., Spark, TensorFlow)?**  

Let me know how you'd like to proceed! 🚀🎬

<br/>
<br/>

# **14. Designing a Collaborative Document Editing Platform Using Cassandra🔥🔥**  

### **🔹 Key Requirements:**  
✔ **Efficiently store and retrieve document changes instead of the full document.**  
✔ **Support real-time collaboration with multiple users editing simultaneously.**  
✔ **Ensure conflict resolution and ordering of changes.**  
✔ **Scale to millions of documents and concurrent users.**  

---

## **📌 1. Table: Storing Document Changes (Operational Transformation Model)**  
🔹 _Stores a log of changes instead of full document content._  

```cql
CREATE TABLE document_changes (
    doc_id UUID,  
    change_time timestamp,  
    user_id UUID,  
    operation text,  -- "INSERT", "DELETE", "UPDATE"
    position int,  -- Character/word position in the document
    content text,  
    PRIMARY KEY (doc_id, change_time)
) WITH CLUSTERING ORDER BY (change_time ASC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `doc_id` → Groups all changes for a specific document.  
✔ **Clustering Key:** `change_time ASC` → Ensures changes are **applied in chronological order**.  
✔ **Operational Model:** Stores **insertions, deletions, updates** instead of entire document snapshots.  

### **📌 Query Examples:**  
✔ **Retrieve All Changes for a Document in Order**  
```cql
SELECT * FROM document_changes WHERE doc_id = 5678;
```

✔ **Insert a New Edit Operation**  
```cql
INSERT INTO document_changes (doc_id, change_time, user_id, operation, position, content) 
VALUES (5678, toTimestamp(now()), 1234, 'INSERT', 10, 'Hello');
```

✔ **Delete a Word from a Document**  
```cql
INSERT INTO document_changes (doc_id, change_time, user_id, operation, position, content) 
VALUES (5678, toTimestamp(now()), 1234, 'DELETE', 15, '');
```

✔ **Retrieve Changes Within a Specific Time Window**  
```cql
SELECT * FROM document_changes WHERE doc_id = 5678 
AND change_time >= '2024-07-10 12:00:00' AND change_time <= '2024-07-10 14:00:00';
```

---

## **📌 2. Table: Storing Document Snapshots (For Faster Reads)**  
🔹 _Periodically stores the full document state to avoid reconstructing from changes._  

```cql
CREATE TABLE document_snapshots (
    doc_id UUID PRIMARY KEY,  
    last_modified timestamp,  
    content text
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `doc_id` → Each document has a **single snapshot** entry.  
✔ **Avoids reconstructing full document from changes** by storing periodic snapshots.  

### **📌 Query Examples:**  
✔ **Retrieve the Latest Document Content**  
```cql
SELECT content FROM document_snapshots WHERE doc_id = 5678;
```

✔ **Insert a New Document Snapshot (Periodic Backup)**  
```cql
INSERT INTO document_snapshots (doc_id, last_modified, content) 
VALUES (5678, toTimestamp(now()), 'Final version of the document');
```

✔ **Update a Document Snapshot After Merging Changes**  
```cql
UPDATE document_snapshots 
SET content = 'Updated version of the document', last_modified = toTimestamp(now()) 
WHERE doc_id = 5678;
```

---

## **📌 3. Table: Tracking Active Editors (For Real-Time Collaboration)**  
🔹 _Stores active users currently editing a document._  

```cql
CREATE TABLE document_editors (
    doc_id UUID,  
    user_id UUID,  
    last_activity timestamp,  
    PRIMARY KEY (doc_id, user_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `doc_id` → Groups editors by document.  
✔ **Clustering Key:** `user_id` → Ensures efficient lookups by user.  

### **📌 Query Examples:**  
✔ **Retrieve All Active Users Editing a Document**  
```cql
SELECT * FROM document_editors WHERE doc_id = 5678;
```

✔ **Insert a New Active Editor**  
```cql
INSERT INTO document_editors (doc_id, user_id, last_activity) 
VALUES (5678, 1234, toTimestamp(now()));
```

✔ **Remove User from Editing Session When Inactive**  
```cql
DELETE FROM document_editors WHERE doc_id = 5678 AND user_id = 1234;
```

---

## **📌 4. Table: Tracking User Access Control (Permissions System)**  
🔹 _Manages who can edit or view a document._  

```cql
CREATE TABLE document_permissions (
    doc_id UUID,  
    user_id UUID,  
    permission text,  -- "OWNER", "EDITOR", "VIEWER"
    PRIMARY KEY (doc_id, user_id)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `doc_id` → Stores permissions per document.  
✔ **Clustering Key:** `user_id` → Ensures efficient permission lookups.  

### **📌 Query Examples:**  
✔ **Retrieve a User's Permission on a Document**  
```cql
SELECT permission FROM document_permissions WHERE doc_id = 5678 AND user_id = 1234;
```

✔ **Grant Edit Access to a User**  
```cql
INSERT INTO document_permissions (doc_id, user_id, permission) 
VALUES (5678, 1234, 'EDITOR');
```

✔ **Revoke Access**  
```cql
DELETE FROM document_permissions WHERE doc_id = 5678 AND user_id = 1234;
```

---

## **📌 Summary of Schema Design**
| **Table** | **Partition Key** | **Clustering Key** | **Use Case** |
|-----------|----------------|----------------|-------------|
| **document_changes** | `doc_id` | `change_time` | Stores incremental document edits. |
| **document_snapshots** | `doc_id` | None | Stores full document snapshots for fast access. |
| **document_editors** | `doc_id` | `user_id` | Tracks active users editing a document. |
| **document_permissions** | `doc_id` | `user_id` | Manages user access and permissions. |

---

## **📌 Optimizations for Performance**
✔ **Use `TimeWindowCompactionStrategy (TWCS)` for Efficient Change Storage**  
```cql
ALTER TABLE document_changes 
WITH compaction = { 'class': 'TimeWindowCompactionStrategy', 'compaction_window_size': '1', 'compaction_window_unit': 'HOURS' };
```
🔹 _Groups changes into **hourly SSTables** for efficient retrieval._

✔ **Enable Caching for Frequently Accessed Documents**  
```cql
ALTER TABLE document_snapshots WITH caching = { 'keys': 'ALL', 'rows_per_partition': '100' };
```
🔹 _Speeds up **retrieval of full document snapshots.**_

✔ **Use TTL for Auto-Deleting Old Change Logs (Retain 30 Days of Changes)**  
```cql
INSERT INTO document_changes (doc_id, change_time, user_id, operation, position, content) 
VALUES (5678, toTimestamp(now()), 1234, 'INSERT', 10, 'Hello') USING TTL 2592000;
```
🔹 _Deletes **old changes automatically after 30 days.**_

---

## **📌 Next Steps**
Would you like me to:  
✔ **Add conflict resolution mechanisms for concurrent edits?**  
✔ **Provide real-world CQL scripts for testing document collaboration?**  
✔ **Suggest best practices for integrating Cassandra with WebSockets for live updates?**  

Let me know how you'd like to proceed! 🚀📄

<br/>
<br/>

# **15. Designing a Time-Series Data Model for an Application Monitoring System in Cassandra🔥🔥**  

### **🔹 Key Requirements:**  
✔ **Efficiently store and query system metrics collected every minute.**  
✔ **Allow fast retrieval of metrics over a time range for analysis.**  
✔ **Scale to handle multiple systems and millions of data points.**  
✔ **Ensure efficient compaction strategy to optimize storage and retrieval.**  

---

## **📌 1. Table: Storing System Metrics (Time-Series Model)**  
🔹 _Stores time-series metrics for each system and metric type._  

```cql
CREATE TABLE metrics (
    system_id UUID,  
    metric_name text,  
    recorded_at timestamp,  
    value decimal,  
    PRIMARY KEY ((system_id, metric_name), recorded_at)
) WITH CLUSTERING ORDER BY (recorded_at DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `(system_id, metric_name)` → Groups metrics by system and metric type.  
✔ **Clustering Key:** `recorded_at DESC` → Ensures **latest metrics appear first**.  
✔ **Wide-Row Model:** Stores **all readings for a metric efficiently**.  

### **📌 Query Examples:**  
✔ **Retrieve the Latest Value for a Specific Metric on a System**  
```cql
SELECT * FROM metrics WHERE system_id = 1234 AND metric_name = 'CPU_Usage' LIMIT 1;
```

✔ **Retrieve Metrics for a Specific Time Range**  
```cql
SELECT * FROM metrics WHERE system_id = 1234 AND metric_name = 'Memory_Usage' 
AND recorded_at >= '2024-07-10 12:00:00' AND recorded_at <= '2024-07-10 13:00:00';
```

✔ **Insert a New Metric Reading**  
```cql
INSERT INTO metrics (system_id, metric_name, recorded_at, value) 
VALUES (1234, 'Disk_Usage', toTimestamp(now()), 78.5);
```

✔ **Auto-Delete Old Metrics (Retain Data for 30 Days)**  
```cql
INSERT INTO metrics (system_id, metric_name, recorded_at, value) 
VALUES (1234, 'Network_Throughput', toTimestamp(now()), 500.2) USING TTL 2592000;
```
🔹 _Deletes records **after 30 days (2,592,000 seconds).**_

---

## **📌 2. Table: Aggregated Metrics (Daily Summary for Fast Queries)**  
🔹 _Stores precomputed daily statistics (min, max, avg) for each metric._  

```cql
CREATE TABLE daily_metric_summary (
    system_id UUID,  
    metric_name text,  
    date date,  
    min_value decimal,  
    max_value decimal,  
    avg_value decimal,  
    PRIMARY KEY ((system_id, metric_name), date)
);
```

### **✅ Explanation:**  
✔ **Partition Key:** `(system_id, metric_name)` → Groups summaries by system and metric type.  
✔ **Clustering Key:** `date` → Stores daily summaries in chronological order.  

### **📌 Query Examples:**  
✔ **Retrieve Last 7 Days of CPU Usage Summary**  
```cql
SELECT * FROM daily_metric_summary WHERE system_id = 1234 AND metric_name = 'CPU_Usage' 
AND date >= '2024-07-04' AND date <= '2024-07-10';
```

✔ **Insert a Daily Summary Record**  
```cql
INSERT INTO daily_metric_summary (system_id, metric_name, date, min_value, max_value, avg_value) 
VALUES (1234, 'CPU_Usage', '2024-07-10', 20.5, 95.3, 60.8);
```

---

## **📌 3. Table: Alerting System (Detecting Anomalies in Real-Time)**  
🔹 _Stores alert events when a metric exceeds a threshold._  

```cql
CREATE TABLE metric_alerts (
    system_id UUID,  
    metric_name text,  
    alert_time timestamp,  
    alert_type text,  
    alert_value decimal,  
    message text,  
    PRIMARY KEY ((system_id, metric_name), alert_time)
) WITH CLUSTERING ORDER BY (alert_time DESC);
```

### **✅ Explanation:**  
✔ **Partition Key:** `(system_id, metric_name)` → Tracks alerts per system and metric.  
✔ **Clustering Key:** `alert_time DESC` → Stores recent alerts **first**.  

### **📌 Query Examples:**  
✔ **Retrieve Last 5 Alerts for a System's CPU Usage**  
```cql
SELECT * FROM metric_alerts WHERE system_id = 1234 AND metric_name = 'CPU_Usage' LIMIT 5;
```

✔ **Insert a New Alert When CPU Usage Exceeds 90%**  
```cql
INSERT INTO metric_alerts (system_id, metric_name, alert_time, alert_type, alert_value, message) 
VALUES (1234, 'CPU_Usage', toTimestamp(now()), 'HIGH', 95.3, 'CPU usage exceeded 90%');
```

---

## **📌 4. Choosing the Right Compaction Strategy**  

### **🔹 Best Compaction Strategies for Time-Series Data:**  

| **Strategy** | **Use Case** | **Pros** | **Cons** |
|-------------|-------------|---------|---------|
| **Time-Window Compaction Strategy (TWCS)** | Best for time-series data with frequent writes | Optimized for sequential writes, low read amplification | Slightly higher disk usage |
| **Leveled Compaction Strategy (LCS)** | Best when frequent reads and updates are needed | Low read latency, reduces space overhead | High CPU & disk I/O usage |

### **✅ Recommended Strategy: `TimeWindowCompactionStrategy (TWCS)`**  
```cql
ALTER TABLE metrics 
WITH compaction = { 
    'class': 'TimeWindowCompactionStrategy', 
    'compaction_window_size': '1',  
    'compaction_window_unit': 'DAYS'
};
```
🔹 _Groups metric data into **daily SSTables** for efficient storage._  

---

## **📌 Summary of Schema Design**
| **Table** | **Partition Key** | **Clustering Key** | **Use Case** |
|-----------|----------------|----------------|-------------|
| **metrics** | `(system_id, metric_name)` | `recorded_at` | Stores real-time system metrics. |
| **daily_metric_summary** | `(system_id, metric_name)` | `date` | Stores aggregated daily system metrics. |
| **metric_alerts** | `(system_id, metric_name)` | `alert_time` | Tracks anomalies in system metrics. |

---

## **📌 Optimizations for Performance**
✔ **Use `TWCS` for Efficient Metric Storage**  
```cql
ALTER TABLE metrics 
WITH compaction = { 'class': 'TimeWindowCompactionStrategy', 'compaction_window_size': '1', 'compaction_window_unit': 'DAYS' };
```
🔹 _Optimizes storage by **grouping daily metrics** into SSTables._

✔ **Enable Caching for Frequently Accessed Aggregates**  
```cql
ALTER TABLE daily_metric_summary WITH caching = { 'keys': 'ALL', 'rows_per_partition': '100' };
```
🔹 _Improves retrieval speed for **historical metric trends**._

✔ **Use TTL for Auto-Deleting Old Data**  
```cql
INSERT INTO metrics (system_id, metric_name, recorded_at, value) 
VALUES (1234, 'CPU_Usage', toTimestamp(now()), 65.5) USING TTL 2592000;
```
🔹 _Deletes **old metrics automatically after 30 days.**_

---

## **📌 Next Steps**
Would you like me to:  
✔ **Add real-time anomaly detection and alerting logic?**  
✔ **Provide real-world CQL scripts for simulating system monitoring?**  
✔ **Suggest best practices for integrating Cassandra with real-time streaming tools (e.g., Apache Kafka, Spark)?**  
