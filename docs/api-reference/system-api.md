---
id: system-api
title: System API Reference
sidebar_position: 5
---

# System API Reference

This document outlines the System REST API endpoints for the Trip Otter Next.js backend, including request parameters, payload shapes, and responses.

## Analytics (`/api/analytics`)

Fetches system-wide analytics data, including yearly and monthly counts for posts, comments, likes, and users.

- **Methods Supported:** `GET`, `OPTIONS`
- **Authentication:** Not required

### `GET /api/analytics`

#### Response Examples

**Success (200 OK)**
```json
{
  "status": 200,
  "data": {
    "yearly": {
      "postsThisYear": 150,
      "commentsThisYear": 300,
      "likesThisYear": 1200,
      "usersJoinedThisYear": 50
    },
    "monthly": [
      {
        "month": "January",
        "posts": 10,
        "comments": 20,
        "likes": 100,
        "usersJoined": 5
      }
    ]
  }
}
```

**Error (500 Internal Server Error)**
```json
{
  "status": 500,
  "data": {}
}
```

---

## Health (`/api/health`)

Basic application health check that also pings the database.

- **Methods Supported:** `GET`, `OPTIONS`
- **Authentication:** Not required

### `GET /api/health`

#### Response Example

**Success (200 OK)**
```json
{
  "message": "App running and database hit",
  "status": 200,
  "method": "GET"
}
```

---

## Report (`/api/report`)

Manages user reports against other users, posts, or comments.

- **Methods Supported:** `GET`, `POST`, `PATCH`, `DELETE`, `OPTIONS`
- **Authentication:** :::warning Required
  All methods require an active NextAuth session. Unauthenticated requests return `401 Unauthorized`.
  :::

### `GET /api/report`

| Query Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `id` | `string` | No | ID of a specific report. If omitted, returns all reports matching undefined/null. |

#### Response Example (200 OK)
```json
{
  "message": "Report fetched",
  "status": 200,
  "data": []
}
```

### `POST /api/report`

#### Request Body
```json
{
  "data": {
    "reportedUser": "string",
    "scope": "string",
    "reason": "string",
    "reasonDescription": "string (optional)",
    "relatedComment": "string (optional)",
    "relatedPost": "string (optional)"
  }
}
```

#### Response Examples

**Success (200 OK)**
```json
{
  "message": "Report created",
  "status": 200,
  "data": {
    "profile": { ... },
    "reportResponse": { ... }
  }
}
```

**Error (401 Unauthorized)**
```json
{
  "message": "Unauthorized",
  "status": 401
}
```

**Error (500 Internal Server Error)**
```json
{
  "message": "Internal server error",
  "status": 500
}
```

### `PATCH /api/report`

Updates a report. Currently stubbed to return a 200 response.

| Query Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `id` | `string` | No | ID of the report to update. |

#### Response Example (200 OK)
```json
{
  "message": "Report updated",
  "status": 200
}
```

### `DELETE /api/report`

Deletes a report. Currently stubbed to return a 200 response.

| Query Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `id` | `string` | No | ID of the report to delete. |

#### Response Example (200 OK)
```json
{
  "message": "Report deleted",
  "status": 200
}
```

---

## Search (`/api/search`)

Search operations for users, shops, and hashtags.

- **Methods Supported:** `GET`, `POST`, `OPTIONS`
- **Authentication:** Not required

### `GET /api/search`

| Query Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `profile` | `string` | No | Search term for `fullName` across active users. |
| `group` | `string` | No | Filter for groups (Not implemented). |
| `shop` | `string` | No | Filter for shops (Not implemented). |
| `hashtags` | `string` | No | Search term for `hashtags` in posts. |
| `page` | `number` | No | Page number for pagination. Default `1`. |
| `limit` | `number` | No | Items per page. Default `10`. |

#### Response Example (200 OK)
```json
{
  "message": "Fetched search results",
  "status": 200,
  "method": "GET",
  "data": {
    "users": [],
    "shops": [],
    "hashtags": []
  }
}
```

### `POST /api/search`
*Note: Currently a placeholder endpoint returning "Hello World".*

---

## Users (`/api/users`)

Retrieve and update user profiles.

- **Methods Supported:** `GET`, `POST`, `PATCH`, `OPTIONS`
- **Authentication:**
  - `GET`: Not required.
  - `PATCH`: :::warning Required
    Requires NextAuth session.
    :::

### `GET /api/users`

| Query Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `id` | `string` | Yes | Valid MongoDB ObjectId of the user. |

#### Response Examples

**Success (200 OK)**
```json
{
  "message": "User data retrieved successfully",
  "status": 200,
  "data": {
    "_id": "5f8d0a...",
    "username": "johndoe",
    "fullName": "John Doe",
    "profile": {
      "postsCount": 10,
      "commentsCount": 5,
      "followersCount": 100,
      "followingCount": 50
    }
  }
}
```

**Error (400 Bad Request)**
```json
{
  "message": "User ID is required",
  "status": 400
}
```

**Error (404 Not Found)**
```json
{
  "message": "User not found",
  "status": 404
}
```

### `PATCH /api/users`

Updates the authenticated user's details.

#### Request Body
```json
{
  "username": "new_username",
  "bio": "New bio..."
}
```
### PATCH `/api/users`

:::danger 🚨 CRITICAL: ARBITRARY FIELD INJECTION
**Status:** Unpatched;
This endpoint reads `request.json()` and passes the entire unvalidated object directly into a MongoDB `$set` operator. Because it bypasses Zod schema validation, a malicious actor can inject restricted fields into their own document (e.g., passing `"role": "BUSINESS"` or `"reputation": 9999`).
:::

#### Response Example (200 OK)
```json
{
  "message": "Profile Updated!",
  "status": 200,
  "data": { ... }
}
```

### `POST /api/users`
*Note: Currently a placeholder endpoint returning "Hello World".*

---

## Companion (`/api/companion`)

Fetches suggested users for a given user to follow.

- **Methods Supported:** `GET`, `POST`, `OPTIONS`
- **Authentication:** Not required

### `GET /api/companion`

| Query Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `userId` | `string` | Yes | The ID of the user requesting suggestions. |
| `page` | `number` | No | Page number for pagination. Default `1`. |
| `limit` | `number` | No | Items per page. Default `10`. |

#### Response Examples

**Success (200 OK)**
```json
{
  "message": "Hello World",
  "status": 200,
  "data": [
    {
      "_id": "603d...",
      "user": {
        "_id": "603d...",
        "fullName": "Jane Doe",
        "username": "janedoe",
        "bio": "Travel enthusiast",
        "location": "NY",
        "role": "user"
      }
    }
  ]
}
```

**Error (400 Bad Request)**
```json
{
  "error": "userId is required"
}
```

### `POST /api/companion`
*Note: Currently a placeholder endpoint returning "Hello World".*
