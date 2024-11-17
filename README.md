# Social Media Platform Design with MongoDB Schema and API Specification

This repository contains the design and implementation for a basic social media platform backend using MongoDB. The platform supports user registration, friend management, post creation, commenting, and a personalized content feed.  

The implementation includes the following features:
1. User registration and authentication.
2. Friend request management (send, accept, reject).
3. Post creation and commenting.
4. A feed API to fetch personalized content as per requirements.

---

## Table of Contents
- [Features](#features)
- [Schema Design](#schema-design)
- [API Endpoints](#api-endpoints)

---

## Features
- **User Management**: Register and store user details.
- **Friend Management**:
  - Send and receive friend requests.
  - Accept or reject friend requests.
- **Content Management**:
  - Create text-only posts.
  - Comment on posts made by friends.
- **Feed Generation**:
  - Display posts from friends.
  - Display posts from non-friends if a friend has commented on them.

---

## Schema Design
The platform uses MongoDB for its schema-less and scalable capabilities. Below is the proposed schema for different collections:

### 1. Users Collection
Stores user details and their relationships and also handles freind requests.

```json
{
  "_id": "ObjectId",  // Unique identifier for the user
  "username": "string",  // Unique username
  "email": "string",  // Unique email address
  "passwordHash": "string",  // Hashed password
  "profile": {
    "fullName": "string",
    "bio": "string",
    "profilePicture": "string",  // URL to the profile picture
    "createdAt": "Date"
  },
  "friendIds": ["ObjectId"],  // List of User IDs for accepted friends
  "pendingRequests": {
    "sent": ["ObjectId"],  // User IDs of pending sent requests
    "received": ["ObjectId"]  // User IDs of pending received requests
  }
}

```

### 2. Posts Collection
Stores User's posts.

```json
{
  "_id": "ObjectId",  // Unique identifier for the post
  "userId": "ObjectId",  // User ID of the creator
  "content": "string",  // Text-only content of the post
  "createdAt": "Date",
  "comments": [  // Embedded comments
    {
      "commentId": "ObjectId",  // Unique identifier for the comment
      "userId": "ObjectId",  // User ID of the commenter
      "content": "string",  // Text-only comment content
      "createdAt": "Date"
    }
  ]
}

```

---

## API Endpoints
The following APIs can be used to fulfill the stated requirements:

### 1. Get Posts by friends
This API will allow a user to fetch only the posts created by their friends.

Endpoint: `GET /api/posts/friends/{userId}`

URL Parameters: 
* `userId`: The id of the user requesting the posts.

Request Headers:
* `Authorization`: Bearer token for authenticated user.

Response:
```json
{
  "posts": [ //Array of posts created by user's freinds.
    {
      "postId": "111", //Unique id of post
      "userId": "67890", //userId of the user who created the post
      "author": "Alice",
      "content": "Having a great day!",
      "createdAt": "2024-11-17T10:00:00Z",
      "comments": [
        {
          "commentId": "201",
          "userId": "12345",
          "content": "Awesome!",
          "createdAt": "2024-11-17T11:00:00Z"
        }
      ]
    },
    {
      "postId": "112",
      "userId": "67891",
      "author": "Bob",
      "content": "Just finished reading a book!",
      "createdAt": "2024-11-16T08:30:00Z",
      "comments": []
    }
  ]
}
```

### 2. Get Friends' Post Visibility Check
In the background, when retrieving posts, the system will need to check the friendship between the user and the post's author to filter only posts created by friends. This can be done by validating the relationship in the Friendships table or collection.

Example Query (MongoDB):

To fetch posts created by the user's friends, you would perform the following steps:

a) Get the list of friends

b) Filter posts to include only those created by users in the friends list
```js
// Pseudo code: MongoDB query to get posts by friends
db.posts.find({
  userId: { $in: userFriendsList }
})
```

### 3. Get Posts by friends and non-friends with friend comments
This API will fetch posts where:
* Posts created by a user's friends are always shown.
* Posts created by non-friends will be shown only if at least one of the user's friends has commented on them.

Endpoint: `GET /api/posts/extended-feed/{userId}`

URL Parameters:
* `userId`: The ID of the user requesting the posts (authenticated user).

Request Header:
* `Authoroization`:Bearer token for the authenticated user.

Database Queries(Example):
```js
// Get user's friends
let userFriendsList = db.users.findOne({ _id: userId }).friends;

// Fetch posts created by friends and non-friends with friend comments
let posts = db.posts.find({
  $or: [
    { userId: { $in: userFriendsList } }, // Posts by friends
    {
      comments: {
        $elemMatch: {
          userId: { $in: userFriendsList } // At least one comment by a friend
        }
      }
    }
  ]
});
```

Response Format:
```json
{
  "posts": [
    {
      "postId": "111",
      "userId": "67890",
      "author": "Alice",
      "content": "Having a great day!",
      "createdAt": "2024-11-17T10:00:00Z",
      "comments": [
        {
          "commentId": "201",
          "userId": "12345",
          "content": "Awesome!",
          "createdAt": "2024-11-17T11:00:00Z"
        }
      ]
    },
    {
      "postId": "112",
      "userId": "98765",
      "author": "Charlie",
      "content": "Check out this cool book I just read!",
      "createdAt": "2024-11-16T08:30:00Z",
      "comments": [
        {
          "commentId": "202",
          "userId": "12345",
          "content": "Looks interesting!",
          "createdAt": "2024-11-16T09:00:00Z"
        }
      ]
    },
    {
      "postId": "113",
      "userId": "67892",
      "author": "David",
      "content": "Loving the weather today!",
      "createdAt": "2024-11-15T07:45:00Z",
      "comments": []
    }
  ]
}
```
