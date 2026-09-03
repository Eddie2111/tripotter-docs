---
id: tribes-api
title: Tribes API
sidebar_position: 3
---

# Tribes API Reference

This document outlines the REST API endpoints for Tribes in the Next.js backend.

## `/api/tribe`

### GET
Retrieves a list of tribes or a specific tribe based on query parameters.

:::info
Requires NextAuth session authentication. Returns 401 if unauthorized.
:::

**Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | The MongoDB ObjectId of the tribe. |
| `serial` | string | The unique serial string of the tribe. |
| `page` | string | Page number for pagination. |
| `limit` | string | Number of items per page. |
| `user` | string | The user ObjectId to fetch created/joined tribes for. |
| `member` | string | Pass `"true"` (with `id`, `page`, `limit`) to fetch tribe members. |
| `posts` | string | Pass `"true"` (with `id`, `page`, `limit`) to fetch tribe posts. |
| `ownership` | string | Enum: `"joined"`, `"created"`, or any other value for "unjoined". Fetches tribes for the current authenticated user. |

**Response Example (200 OK)**
```json
{
  "message": "Get tribes",
  "status": 200,
  "data": []
}
```

### POST
Creates a new tribe.

:::info
Requires NextAuth session authentication. Returns 401 if unauthorized.
:::

**Request Body**
```json
{
  "name": "Mountain Explorers",
  "description": "A group for mountain lovers",
  "category": "COMMUNITY",
  "tags": ["mountains", "hiking"],
  "coverImage": "https://example.com/cover.jpg",
  "profileImage": "https://example.com/profile.jpg",
  "createdBy": "64c8c7f99999999999999999",
  "privacy": "PUBLIC"
}
```
*(Note: `category` defaults to "COMMUNITY", `privacy` defaults to "PUBLIC".)*

**Response Example (200 OK)**
```json
{
  "message": "Created tribe",
  "status": 200,
  "data": {
    "_id": "64c8c7f99999999999999999",
    "name": "Mountain Explorers",
    "serial": "uuid-string-here",
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```

### PATCH
Updates an existing tribe.

:::info
Requires NextAuth session authentication. Returns 401 if unauthorized.
:::

**Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `serial` | string | The unique serial string of the tribe to update. |

**Request Body**
Accepts a partial JSON payload with the fields to update (e.g., `name`, `description`). 

**Response Example (200 OK)**
```json
{
  "message": "Tribe updated",
  "status": 200,
  "data": {
    "_id": "64c8c7f99999999999999999",
    "serial": "uuid-string-here",
    "name": "Updated Name"
  }
}
```

### DELETE
Deletes a tribe.

:::info
Requires NextAuth session authentication. Returns 401 if unauthorized.
:::

**Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `id` | string | **Note:** This must be the unique **serial** string of the tribe, despite the parameter name. |

**Response Example (200 OK)**
```json
{
  "message": "Deleted Tribe",
  "status": 200,
  "data": {
    "_id": "64c8c7f99999999999999999",
    "serial": "uuid-string-here"
  }
}
```

---

## `/api/tribe/join`

### GET
Checks tribe membership and admin status.

:::info
Requires NextAuth session authentication. 
:::

**Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `requestType` | string | Must be `"memberCheck"`. |
| `tribeId` | string | The unique **serial** string of the tribe. |
| `userId` | string | The MongoDB ObjectId of the user. |

**Response Example (200 OK)**
```json
{
  "message": "Checked tribe membership",
  "status": 200,
  "data": {
    "isMember": true,
    "isAdmin": false
  }
}
```

### POST
Joins or leaves a tribe (toggles membership).

:::info
Requires NextAuth session authentication. Returns 401 if unauthorized.
:::

**Request Body**
```json
{
  "tribeId": "64c8c7f99999999999999999",
  "userId": "64c8c7f99999999999999999"
}
```
*(Note: `tribeId` here must be the MongoDB ObjectId, not the serial).*

**Response Example (200 OK)**
```json
{
  "message": "Joined tribe",
  "status": 200,
  "data": {
    "tribe": {
      "_id": "64c8c7f99999999999999999",
      "name": "Mountain Explorers"
    },
    "action": "joined"
  }
}
```

---

## `/api/tribe/search`

### GET
Searches and filters tribes.

:::warning
This endpoint expects a JSON request body on a GET request, which is non-standard and may not be supported by all HTTP clients. Session authentication check is currently commented out.
:::

**Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `searchText` | string | The text to search for in the tribe's `name` using regex. |

**Request Body**
```json
{
  "privacy": "PUBLIC",
  "category": "COMMUNITY",
  "tags": ["hiking"]
}
```

**Response Example (200 OK)**
```json
{
  "message": "get searched tribes",
  "status": 200,
  "data": [
    {
      "_id": "64c8c7f99999999999999999",
      "name": "Mountain Explorers",
      "category": "COMMUNITY"
    }
  ]
}
```

### POST
Searches and filters tribes (includes rate limiting and omits `users`, `posts`, and `updatedAt` from results).

:::warning
Session authentication check is currently commented out.
:::

**Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `searchText` | string | The text to search for in the tribe's `name` using regex. |

**Request Body**
```json
{
  "privacy": "PUBLIC",
  "category": "COMMUNITY",
  "tags": ["hiking"]
}
```

**Response Example (200 OK)**
```json
{
  "message": "get searched tribes",
  "status": 200,
  "data": [
    {
      "_id": "64c8c7f99999999999999999",
      "name": "Mountain Explorers",
      "category": "COMMUNITY"
    }
  ]
}
```
