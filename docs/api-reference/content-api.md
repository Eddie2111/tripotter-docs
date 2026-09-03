---
id: content-api
title: Content API
sidebar_position: 4
---

# Content API Reference

This document outlines the Content APIs available in the Trip Otter backend, used for managing feeds, journeys, locations, media, posts, and suggestions.

## `/api/feed`
Retrieves a paginated list of public feed posts.

:::info Authentication
This endpoint does not require authentication.
:::

### `GET`

**Query Parameters**

| Parameter   | Type     | Default | Description |
| ----------- | -------- | ------- | ----------- |
| `id`        | string   | `null`  | Profile ID for personalized feed (optional). |
| `versionId` | string   | `null`  | Set to `"v2"` for v2 testing. |
| `page`      | number   | `1`     | Page number for pagination. |
| `limit`     | number   | `10`    | Number of items per page. |

**Success (200 OK)**
```json
{
  "message": "Received feed data",
  "status": 200,
  "data": [],
  "pagination": {
    "currentPage": 1,
    "postsPerPage": 10,
    "totalPosts": 100,
    "totalPages": 10,
    "hasMore": true
  }
}
```

**Error (500 Internal Server Error)**
```json
{
  "message": "Failed to load posts: <error message>",
  "status": 500,
  "data": [],
  "pagination": {
    "currentPage": 1,
    "postsPerPage": 10,
    "totalPosts": 0,
    "totalPages": 0,
    "hasMore": false
  }
}
```

---

## `/api/journey`
Manages journey/journal creation and retrieval.

:::warning Authentication
`GET` uses `getServerSession(authOptions)` but lacks strong enforcement. `POST` lacks authentication checks entirely.
:::

### `GET`

**Query Parameters**

| Parameter | Type   | Default | Description |
| --------- | ------ | ------- | ----------- |
| `id`      | string | `null`  | Profile ID. |

**Success (200 OK)**
```json
{
  "message": "get journals",
  "status": 200,
  "method": "GET"
}
```

### `POST`

**Success (200 OK)**
```json
{
  "message": "create journals",
  "status": 200,
  "method": "POST"
}
```

---

## `/api/locations`
Retrieves and creates locations.

:::info Authentication
This endpoint does not require authentication.
:::

### `GET`
Retrieves locations based on a search string.

**Query Parameters**

| Parameter  | Type   | Default | Description |
| ---------- | ------ | ------- | ----------- |
| `location` | string | `null`  | Location search string. Empty string returns empty data. |

**Success (200 OK)**
```json
{
  "message": "Got all locations",
  "status": 200,
  "data": [
    {
      "_id": "64...1a",
      "id": "loc123",
      "division_id": "div1",
      "division_name": "Dhaka",
      "district_name": "Dhaka",
      "Location": "Gulshan",
      "lat": "23.79",
      "lng": "90.41",
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  ]
}
```

**Error (500 Internal Server Error)**
```json
{
  "message": "Failed to retrieve location(s)",
  "status": 500,
  "error": "<error message>"
}
```

### `POST`
Creates a new location.

**Request Body**
```json
{
  "id": "loc123",
  "division_id": "div1",
  "division_name": "Dhaka",
  "district_name": "Dhaka",
  "Location": "Gulshan",
  "lat": "23.79",
  "lng": "90.41"
}
```

**Success (201 Created)**
```json
{
  "message": "Location created successfully",
  "status": 201,
  "data": { 
    /* Created Location Object */ 
  }
}
```

---

## `/api/media`
Handles uploading, retrieving, and deleting media files (images and videos).

:::warning Authentication
Strictly requires an authenticated NextAuth session for all operations (returns `401 Unauthorized` if missing). Uses `RateLimiter_Middleware`.
:::

### `GET`
Retrieves media details.

**Query Parameters**

| Parameter | Type   | Required | Description |
| --------- | ------ | -------- | ----------- |
| `id`      | string | Yes      | Media ID to retrieve. |

**Success (200 OK)**
```json
{
  "message": "Media retrieved successfully",
  "url": "https://cdn.sanity.io/...",
  "altText": "filename",
  "dimensions": { 
    /* Optional image dimensions */ 
  }
}
```

**Error (404 Not Found)**
```json
{
  "error": "Media not found"
}
```

### `POST`
Uploads a media file. Images are optimized via `sharp`. Videos are limited to 50MB and images to 10MB.

**Request Payload**
Multipart `FormData` containing a `file` field.

**Success (200 OK) - Image**
```json
{
  "message": "Image uploaded, optimized, and stored in Sanity successfully",
  "optimizedSize": 12345,
  "uniqueFilename": "1234-5678.webp",
  "sanityAssetId": "image-...",
  "mediaId": "doc-..."
}
```

**Error (400 Bad Request)**
```json
{
  "error": "File size exceeds 10MB limit."
}
```

### `DELETE`
Deletes a media asset.

**Query Parameters**

| Parameter | Type   | Required | Description |
| --------- | ------ | -------- | ----------- |
| `id`      | string | Yes      | Media ID to delete. |

---

## `/api/posts`
Manages standard posts and journals.

:::info Authentication
No explicit NextAuth session checks. Security relies on explicit passing of `owner` property in payloads.
:::

### `GET`
Retrieves a specific post by ID.

**Query Parameters**

| Parameter | Type   | Required | Description |
| --------- | ------ | -------- | ----------- |
| `postId`  | string | Yes      | Valid MongoDB ObjectId. |

**Success (200 OK)**
```json
{
  "message": "Post retrieved successfully",
  "status": 200,
  "data": { 
    /* Populated post data including stats, comments, and likedBy */ 
  }
}
```

**Error (400 Bad Request)**
```json
{
  "message": "Invalid post ID format",
  "status": 400
}
```

### `POST`
Creates a new post.

**Request Body**
```json
{
  "owner": "64...1a",
  "image": ["url1"],
  "likes": [],
  "caption": "Hello world #travel",
  "location": "Dhaka",
  "comments": [],
  "fromGroup": "64...2b"
}
```

**Success (200 OK)**
```json
{
  "message": "Post uploaded!",
  "status": 200,
  "data": { 
    /* Created post and user update result */ 
  }
}
```

### `PATCH`
Updates a post's caption or location.

**Request Body**
```json
{
  "postId": "64...1a",
  "caption": "New caption",
  "location": "New Location"
}
```

**Success (200 OK)**
```json
{
  "message": "Post updated successfully!",
  "status": 200,
  "data": { 
    /* Updated post object */ 
  }
}
```

### `DELETE`
Deletes a post.

**Query Parameters**

| Parameter | Type   | Required | Description |
| --------- | ------ | -------- | ----------- |
| `id`      | string | Yes      | Valid MongoDB ObjectId. |

**Success (200 OK)**
```json
{
  "message": "Post deleted successfully!",
  "status": 200,
  "data": { 
    /* Deleted post object */ 
  }
}
```

---

## `/api/suggestion`
Retrieves filtered suggestion data.

:::info Authentication
This endpoint does not require authentication.
:::

### `POST`
Retrieves suggestions. Uses a combination of query parameters and POST body for complex filtering.

**Query Parameters**

| Parameter  | Type   | Default | Description |
| ---------- | ------ | ------- | ----------- |
| `profile`  | string | `null`  | Profile filter. |
| `group`    | string | `null`  | Group filter. |
| `shop`     | string | `null`  | Shop filter. |
| `hashtags` | string | `null`  | Hashtag search. |
| `page`     | number | `1`     | Page number. |
| `limit`    | number | `10`    | Results limit. |

**Request Body**
```json
{
  "userId": "64...1a"
}
```

**Success (200 OK)**
```json
{
  "message": "Hello World",
  "status": 200,
  "data": { 
    /* Suggestion Payload */ 
  }
}
```
