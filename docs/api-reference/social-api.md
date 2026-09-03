---
id: social-api
title: Social API Reference
sidebar_position: 2
---

# Social API Reference

This document outlines the REST API endpoints for the Next.js backend, covering followers, reviews, reactions, and comments.

## `/api/followers`

Manages user follower and following associations.

### `GET` /api/followers
Retrieves a list of followers and following for a specific user profile.

**Query Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `profileId` | `string` | Yes | Valid MongoDB ObjectId of the user profile. |

**Response (200 Success)**
```json
{
  "message": "Retrieved successfully",
  "status": 200,
  "data": {
    "followers": [...],
    "following": [...]
  }
}
```

**Response (400 Bad Request)**
```json
{
  "message": "Invalid or missing profile ID",
  "status": 400
}
```

### `POST` /api/followers
Toggles a follow relationship (follow/unfollow).

:::warning Authentication Required
This endpoint requires a valid NextAuth session.
:::

**Request Body**
```json
{
  "targetUserId": "string"
}
```

**Response (200 Success)**
```json
{
  "message": "User followed successfully",
  "status": 200,
  "data": {
    "isFollowing": true,
    "followersCount": 10,
    "followingCount": 5
  }
}
```

### `DELETE` /api/followers
Unfollows a user.

:::warning Authentication Required
This endpoint requires a valid NextAuth session.
:::

**Request Body**
```json
{
  "targetUserId": "string"
}
```

**Response (200 Success)**
```json
{
  "isFollowing": false,
  "followersCount": 9,
  "followingCount": 4
}
```
*(Note: Returns the payload object directly rather than wrapped in a standard `data` envelope)*

---

## `/api/review`

Handles submission, retrieval, and management of user reviews and bug reports.

### `GET` /api/review
Fetches a specific review by ID or a paginated list of all reviews.

**Query Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | `string` | No | ID of the specific review to retrieve. |
| `page` | `number` | No | Pagination page number (default: 1). |
| `limit` | `number` | No | Number of items per page (default: 10). |

**Response (200 Success)**
```json
{
  "message": "Recieved data",
  "status": 200,
  "data": [...]
}
```

### `POST` /api/review
Creates a new review or bug report.

:::danger Missing Authentication
This endpoint currently lacks a NextAuth session check and accepts unauthenticated submissions.
:::

**Request Body**
```json
{
  "user": "string (ObjectId)",
  "type": "REVIEW",
  "scope": "USER_EXPERIENCE",
  "review": "string",
  "title": "string",
  "description": "string",
  "media": [{"url": "string"}]
}
```

**Response (200 Success)**
```json
{
  "message": "review submitted",
  "status": 200,
  "data": { ... }
}
```

### `PATCH` /api/review
Updates an existing review.

:::danger Missing Authentication
This endpoint currently lacks a NextAuth session check.
:::

**Query Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | `string` | Yes | The ID of the review to update. |

**Request Body**
Partial update object following the POST body schema.

**Response (200 Success)**
```json
{
  "message": "Review updated",
  "status": 200,
  "data": { ... }
}
```

### `DELETE` /api/review
Deletes a review.

:::danger Missing Authentication
This endpoint currently lacks a NextAuth session check.
:::

**Query Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | `string` | Yes | The ID of the review to delete. |

**Response (200 Success)**
```json
{
  "message": "Review deleted",
  "status": 200,
  "data": { ... }
}
```

---

## `/api/reaction`

Manages like functionality on posts.

### `GET` /api/reaction
Retrieves all likes for a given post.

**Query Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postId` | `string` | Yes | The ID of the post. |

**Response (200 Success)**
```json
{
  "message": "Retrieved likes",
  "status": 200,
  "data": [...]
}
```

### `POST` /api/reaction
Toggles a like on a post.

:::warning Authentication Required
This endpoint requires a valid NextAuth session.
:::

**Request Body**
```json
{
  "post": "string (ObjectId)"
}
```

**Response (200 Success)**
```json
{
  "message": "Post liked successfully",
  "status": 200,
  "data": {
    "isLiked": true,
    "likeCount": 5,
    "likeId": "string"
  }
}
```

### `DELETE` /api/reaction
Unlikes a post.

:::warning Authentication Required
This endpoint requires a valid NextAuth session.
:::

**Request Body**
```json
{
  "postId": "string (ObjectId)"
}
```
*(Note: payload uses `postId` instead of `post`)*

**Response (200 Success)**
```json
{
  "message": "Post unliked",
  "status": 200,
  "data": { ... }
}
```

---

## `/api/comment`

Manages comments on posts.

### `GET` /api/comment
Retrieves comments for a specific post.

**Query Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postId` | `string` | Yes | The ID of the post. |

**Response (200 Success)**
```json
{
  "message": "Retrieved comments",
  "status": 200,
  "data": [...]
}
```

### `POST` /api/comment
Creates a new comment on a post.

:::warning Authentication Required
This endpoint requires a valid NextAuth session.
:::

**Request Body**
```json
{
  "post": "string (ObjectId)",
  "content": "string"
}
```

**Response (200 Success)**
```json
{
  "message": "Posted comment",
  "status": 200,
  "data": { ... }
}
```

### `PATCH` /api/comment
Updates an existing comment.

:::warning Authentication Required
This endpoint requires a valid NextAuth session.
:::

**Request Body**
```json
{
  "id": "string (ObjectId)",
  "content": "string"
}
```

**Response (200 Success)**
```json
{
  "message": "Comment updated",
  "status": 200,
  "data": { ... }
}
```

### `DELETE` /api/comment
Deletes a comment.

:::warning Authentication Required
This endpoint requires a valid NextAuth session.
:::

**Query Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | `string` | Yes | The ID of the comment to delete. |

**Response (200 Success)**
```json
{
  "message": "Comment deleted",
  "status": 200,
  "data": { ... }
}
```
