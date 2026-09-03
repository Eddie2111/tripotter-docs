---
id: auth-api
title: Authentication API
sidebar_position: 1
---

# Authentication API

This document details the REST API endpoints used for user authentication, signup, and verification in Trip Otter.

:::info Authentication Requirements
The endpoints documented here do not require an active NextAuth session (`getServerSession(authOptions)`) as they are primarily used for unauthenticated users signing up or verifying their accounts.
:::

---

## Signup Routes (`/api/auth/signup`)

### POST `/api/auth/signup`

Creates a new user account.

#### Request Body

The request body must be a JSON object conforming to the following structure:

```json
{
  "fullName": "Jane Doe",
  "username": "janedoe123",
  "email": "jane@example.com",
  "password": "StrongPassword1!",
  "confirmPassword": "StrongPassword1!",
  "agreeToTerms": true
}
```

#### Response (200 Success)

```json
{
  "message": "Sign-up successful",
  "data": {
    "fullName": "Jane Doe",
    "username": "janedoe123",
    "email": "jane@example.com",
    "password": "StrongPassword1!",
    "confirmPassword": "StrongPassword1!",
    "agreeToTerms": true
  },
  "status": 200
}
```

#### Response (400 Validation Error)

```json
{
  "message": "Validation failed",
  "errors": [
    {
      "code": "invalid_type",
      "expected": "string",
      "received": "undefined",
      "path": ["email"],
      "message": "Required"
    }
  ],
  "status": 400
}
```

#### Response (500 Internal Server Error)

```json
{
  "message": "Internal server error",
  "status": 500
}
```

---

### PATCH `/api/auth/signup`

:::danger 🚨 CRITICAL ZERO-DAY: ACCOUNT TAKEOVER
**Status:** Unpatched
This endpoint completely lacks `getServerSession` authorization and old-password validation. It accepts a raw `email` and `password` payload and blindly hashes and saves it. **An attacker can pass any user's email into this payload to instantly overwrite their password and hijack the account.** 
:::

#### Request Body

```json
{
  "email": "jane@example.com",
  "password": "NewStrongPassword1!"
}
```

#### Response (200 Success)

```json
{
  "message": "Password changed",
  "status": 200
}
```

#### Response (500 Internal Server Error)

```json
{
  "message": "Internal server error",
  "status": 500
}
```

---

### GET `/api/auth/signup`

Returns a basic health-check message.

#### Response (200 Success)

```json
{
  "message": "Hello World",
  "status": 200,
  "method": "GET"
}
```

---

## Verification Routes (`/api/auth/verification`)

### GET `/api/auth/verification`

Verifies a user's email or account using a JWT token.

#### Query Parameters

| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `id` | `string` | Yes | A JWT string used for verification. |

#### Response (200 Success)

Returns the verified user's profile details.

```json
{
  "message": "user retrieved",
  "status": 200,
  "data": {
    "email": "jane@example.com",
    "fullName": "Jane Doe",
    "serial": "123e4567-e89b-12d3-a456-426614174000",
    "profileImage": "https://example.com/profile.jpg",
    "location": "Earth"
  }
}
```

#### Response (400 Missing Token)

Returns a raw string response.

```text
Provide your token please
```

#### Response (400 Verification Failed / User Not Found)

```json
{
  "message": "user not verified",
  "status": 400,
  "data": null
}
```
*(Note: Message can also be `"user not retrieved"` if the user is not found in the database)*

---

### POST `/api/auth/verification`

Returns a basic health-check message.

#### Response (200 Success)

```json
{
  "message": "Hello World",
  "status": 200,
  "method": "POST"
}
```
