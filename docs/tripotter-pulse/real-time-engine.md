---
id: real-time-architecture
title: Real-Time WebSocket Architecture
sidebar_label: Real-Time Architecture
description: Documentation of the WebSocket lifecycle, events, and infrastructure in TripOtter.
---

# Real-Time WebSocket Architecture

TripOtter utilizes a robust, real-time WebSocket architecture to power features like private messaging, group chats, global chats, and live notifications. The system is split between a **Next.js frontend** hook (`useWebsocket.ts`) and a **NestJS backend gateway** (`chat.gateway.ts`), with an underlying infrastructure utilizing MongoDB and Redis.

---

## 🔄 Connection Lifecycle

The WebSocket connection lifecycle is managed in a stateful manner, relying on specific authentication events to establish a user's presence.

### 1. Client Initialization

On the frontend, the connection is established using the custom `useWebsocket` hook (`lib/useWebsocket.ts`).

- It connects exclusively via the `['websocket']` transport to bypass long-polling.
- It uses the `WS_BASE_URL` environment variable.
- It passes an `authorization` payload (currently structured to pass a Bearer token).

### 2. Socket Connection (`handleConnection`)

When the Socket.IO client connects to the `/chat` namespace, the backend `ChatGateway` triggers `handleConnection`.

- The server immediately emits the `onlineUsers` event to the newly connected client, providing a snapshot of currently active users.

### 3. User Authentication (`userLogin`)

Connecting to the socket is not enough to send messages; the client **must** explicitly authenticate by emitting a `userLogin` event with their `userId` and `username`.

- **Validation**: The gateway verifies the user against the database via `ChatsService.getUser()`.
- **Duplicate Session Handling**: If the user is already logged in elsewhere, the server sends a `forceDisconnect` event to the old socket ID to maintain a single active session.
- **State Management**: The server maps the user ID to the socket ID in memory (`users`, `userSocketMap`, `socketUserMap`).
- **Room Joining**: The gateway fetches all groups the user belongs to and automatically joins the socket to those specific Socket.IO rooms.
- **Broadcast**: Emits `loginSuccess` to the user, and broadcasts `userOnline` and the updated `onlineUsers` list to everyone.

### 4. Disconnection (`handleDisconnect`)

When a user disconnects:

- The gateway removes the socket mapping.
- The user's `isOnline` status is set to `false`.
- A `userOffline` event is broadcasted to all connected clients.

---

## 📡 Event Dictionary

Below is the complete dictionary of WebSocket events handled by the `/chat` namespace.

### Authentication & Presence Events

| Event Name        |      Direction      | Description                                                     |
| :---------------- | :-----------------: | :-------------------------------------------------------------- |
| `userLogin`       | **Client ➔ Server** | Authenticates the socket. Payload: `{ userId, username }`.      |
| `loginSuccess`    | **Server ➔ Client** | Acknowledges successful login. Payload: `{ userId, username }`. |
| `loginFailure`    | **Server ➔ Client** | Emitted if credentials/user validation fails.                   |
| `forceDisconnect` | **Server ➔ Client** | Emitted to an old socket if the user logs in from a new device. |
| `getOnlineUsers`  | **Client ➔ Server** | Requests the current list of online users.                      |
| `onlineUsers`     | **Server ➔ Client** | Broadcasts the full array of online users.                      |
| `userOnline`      | **Server ➔ Client** | Broadcasts that a specific user has come online.                |
| `userOffline`     | **Server ➔ Client** | Broadcasts that a specific user has gone offline.               |

### Private Messaging Events

| Event Name            |      Direction      | Description                                                          |
| :-------------------- | :-----------------: | :------------------------------------------------------------------- |
| `sendPrivateMessage`  | **Client ➔ Server** | Sends a DM. Payload: `{ recipientId, content }`.                     |
| `privateMessage`      | **Server ➔ Client** | Received by the recipient containing the new message.                |
| `privateMessageSent`  | **Server ➔ Client** | Received by the sender acknowledging successful delivery.            |
| `getConversation`     | **Client ➔ Server** | Requests chat history. Payload: `{ recipientId }`.                   |
| `conversationHistory` | **Server ➔ Client** | Returns previous messages. Payload: `{ recipientId, messages: [] }`. |

### Group Chat Events

| Event Name         |      Direction      | Description                                                          |
| :----------------- | :-----------------: | :------------------------------------------------------------------- |
| `createGroup`      | **Client ➔ Server** | Creates a new group. Payload: `{ groupName, memberIds }`.            |
| `groupCreated`     | **Server ➔ Client** | Emitted to all members of the newly created group. Payload: `{ id, name, members, createdBy, createdAt }`. |
| `joinGroup`        | **Client ➔ Server** | Adds a user to a group. Payload: `{ groupId }`.                      |
| `userJoinedGroup`  | **Server ➔ Client** | Broadcasts to the group room that a user joined. Payload: `{ groupId, userId, username }`. |
| `leaveGroup`       | **Client ➔ Server** | Removes a user from a group. Payload: `{ groupId }`.                 |
| `userLeftGroup`    | **Server ➔ Client** | Broadcasts to the group room that a user left. Payload: `{ groupId, userId, username }`. |
| `getGroupsofUser`  | **Client ➔ Server** | Requests all groups the authenticated user belongs to.               |
| `userGroups`       | **Server ➔ Client** | Returns an array of the user's groups.                               |
| `sendGroupMessage` | **Client ➔ Server** | Sends a message to a group. Payload: `{ groupId, content }`.         |
| `groupMessage`     | **Server ➔ Client** | Broadcasts a message to all sockets in the group room.               |
| `getGroupHistory`  | **Client ➔ Server** | Requests previous messages for a group. Payload: `{ groupId }`.      |
| `groupHistory`     | **Server ➔ Client** | Returns group message history. Payload: `{ groupId, messages: [] }`. |

### Global Chat Events

| Event Name              |      Direction      | Description                                                 |
| :---------------------- | :-----------------: | :---------------------------------------------------------- |
| `sendGlobalMessage`     | **Client ➔ Server** | Sends a message to the global feed. Payload: `{ content }`. |
| `globalMessage`         | **Server ➔ Client** | Broadcasts a global message to all connected clients.       |
| `getGlobalMessages`     | **Client ➔ Server** | Requests the recent global chat history.                    |
| `globalMessagesHistory` | **Server ➔ Client** | Returns the global message history array.                   |

### Message Mutation Events

| Event Name                |      Direction      | Description                                               |
| :------------------------ | :-----------------: | :-------------------------------------------------------- |
| `updateMessage`           | **Client ➔ Server** | Edits a message. Payload: `{ messageSerial, content }`.   |
| `messageUpdated`          | **Server ➔ Client** | Broadcasts the updated message document.                  |
| `deleteMessage`           | **Client ➔ Server** | Deletes a specific message. Payload: `{ messageSerial }`. |
| `messageDeleted`          | **Server ➔ Client** | Broadcasts the ID of the deleted message. Payload: `{ messageSerial, groupId? }`. |
| `deleteGroupChatHistory`  | **Client ➔ Server** | Clears all messages in a group (Admin/Creator only).      |
| `groupChatHistoryCleared` | **Server ➔ Client** | Notifies group members that history was cleared. Payload: `{ groupId, clearedBy }`. |

---

## 🧠 The Role of Redis in the Architecture

According to the system architecture (`Pulse <--> Pub/Sub Redis :6379`), Redis plays a critical role in the backend infrastructure, specifically within the `tripotter-pulse` microservice.

### Current Implementation (Background Jobs)

Currently, Redis is primarily utilized by **BullMQ** to offload heavy asynchronous processing from the main event loop.

- **Trending Queue (`trending.processor.ts`)**: Redis tracks metrics and runs scheduled jobs (every 6 hours) to calculate trending posts based on aggregate likes and comments.
- **History Queue (`history.processor.ts`)**: Redis queues user actions (likes, comments, profile visits, searches) so the main WebSocket/HTTP threads are not blocked during database writes.

### Real-Time Scalability (Architectural Context)

While `chat.gateway.ts` currently manages active user status in-memory (`private users: Map<string, User> = new Map();`), **Redis Pub/Sub** is the designated architectural path for horizontal scaling.

As TripOtter grows and requires multiple instances of the `tripotter-pulse` service:

1. The in-memory Maps will be inadequate, as users connected to `Node A` won't see users connected to `Node B`.
2. Implementing the **Socket.IO Redis Adapter** will allow `Node A` to publish a `sendPrivateMessage` event to Redis, which will then route it to `Node B` where the recipient's socket actually resides.
3. User online status will be transitioned from local memory to Redis Key-Value pairs with TTLs (Time To Live).

---

## 💻 Client-Side Integration

The frontend seamlessly integrates with this architecture using a custom hook wrapper around `socket.io-client`.

```typescript title="trip-otter-dev/lib/useWebsocket.ts"
export const useWebsocket = ({
  path,
  shouldAuthenticate = true,
  autoConnect = false,
}) => {
  const socket = useMemo(() => {
    // Note: Ensure access_token is dynamically fetched from session in production
    const accessToken = "access_token";

    return io(WS_BASE_URL, {
      autoConnect,
      transports: ["websocket"], // Enforces strict WS to avoid polling overhead
      path,
      auth: shouldAuthenticate
        ? { authorization: `Bearer ${accessToken}` }
        : undefined,
    });
  }, [autoConnect, path, shouldAuthenticate]);

  return socket;
};
```

:::warning Authentication Note
The useWebsocket hook currently uses a hardcoded "access_token" string. For production, ensure this is hooked up to NextAuth's session token to properly authenticate initial WebSocket handshakes.
:::

### System Error Events (Chat Namespace)

| Event Name |      Direction      | Description                                                                                                                        |
| :--------- | :-----------------: | :--------------------------------------------------------------------------------------------------------------------------------- |
| `error`    | **Server ➔ Client** | Catch-all event emitted when an operation fails (e.g., "User not authenticated", "Group not found", "Not a member of this group"). |

---

## 🔔 Notifications Architecture (Parallel WebSocket Service)

While the `/chat` namespace handles messaging, TripOtter uses a separate WebSocket namespace at `/notification` to handle real-time alerts (Likes, Comments, Follows, Reports). This separation of concerns prevents chat traffic from bottlenecking system notifications.

### Connection Initialization

The frontend connects to the notifications namespace using the same custom hook, but points to a different path:

```tsx
const socket = useWebsocket({
  path: "/notification",
  shouldAuthenticate: true,
  autoConnect: true,
});
```

### Notifications Event Dictionary

| Event Name             |      Direction      | Description                                                                                                        |
| :--------------------- | :-----------------: | :----------------------------------------------------------------------------------------------------------------- |
| `createNotification`   | **Client ➔ Server** | Dispatched when a user performs an action (likes, comments, follows). Payload: `NotificationDocument`. |
| `newNotification`      | **Server ➔ Client** | Broadcasted to the specific receiver letting them know an action occurred. Payload: `NotificationDocument`. |
| `findUserNotification` | **Client ➔ Server** | Requests the active/unread notifications for the authenticated user upon initial load. Payload: `userId` (string). |
| `findAllNotifications` | **Client ➔ Server** | Requests all historical notifications for the user. Payload: `userId` (string). |
| `findOneNotification`  | **Client ➔ Server** | Fetches a specific notification by its ID. Payload: `id` (string). |
| `isNotificationRead`   | **Client ➔ Server** | Marks a specific notification as `isRead: true`. Payload: `id` (string). |
| `updateNotification`   | **Client ➔ Server** | Updates the content or state of an existing notification. Payload: `NotificationDocument`. |
| `notificationUpdated`  | **Server ➔ Client** | Broadcasts the updated notification state back to the client to update the UI. Payload: `NotificationDocument`. |
| `removeNotification`   | **Client ➔ Server** | Deletes a specific notification from the database. Payload: `id` (string). |
