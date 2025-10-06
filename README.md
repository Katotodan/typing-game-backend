# Typing Game — Backend

This is the WebSocket-backed server for the Typing Game project. It provides realtime communication between clients (React front-end) using Socket.IO and also exposes a couple of simple HTTP endpoints.

## Requirements

- Node.js 14+ (Node 16+ recommended)
- npm or yarn

## Install

Install dependencies from the `typing-game-backend` directory:

```bash
npm install
# or
yarn
```

## Running the server

- `npm start` — starts the server with `nodemon` (auto-restarts on file changes)
- `npm run dev` — starts the server with Node.js `--watch` mode

By default the server listens on port 5000. You can override the port with the `PORT` environment variable.

Example:

```bash
PORT=5001 npm start
```

## Environment variables

- `PORT` — port for the server (default: `5000`)
- `CLIENT_URL` — allowed origin for CORS and Socket.IO (default: `http://localhost:3000`)

Create a `.env` file in the backend root to set these values, for example:

```
PORT=5001
CLIENT_URL=http://localhost:3000
```

## HTTP endpoints

- `GET /wake-up` — returns a simple text response `Server is awake` (useful for health checks)
- `GET /getUser` — returns the current list of online users (server memory)

Note: online users are stored in-memory in the `onlineUser` array and will be lost when the server restarts.

## Socket.IO events

The server uses Socket.IO for realtime events. The key events used by the client are described below.

Client to Server events:

- `addUser` (data, callback)
  - data: { username, userImg }
  - Adds a user to the `onlineUser` list (if not already present) and notifies other users with `newUser`.
  - callback receives an acknowledgment object like `{ status: 'ok' }`.

- `request` ([competitors, senderInfo])
  - Sends an incoming call to each competitor socket id in the `competitors` array using the `call` event and creates a room named `${senderInfo.socketId} room`, joining the sender.

- `callAccepted` ([competitorInfo, callerId])
  - Called when a competitor accepts the call; emits `competitor_join` to the caller and joins the competitor to the caller's room.

- `sendCompetitor` (competitors)
  - Forwards the list of competitors back to each competitor via `sendBackCompetitors`.

- `sendPercentage` ([fromSocketId, percentage, roomId])
  - Broadcasts a `receivePercentage` event to the room `${roomId} room` with [fromSocketId, percentage].

- `leaveRoom` (socket_id)
  - Causes the socket to leave any joined rooms that include the suffix `room`.

Server to Client events (emitted by server):

- `newUser` — emitted to other clients when a new user is added. Payload: { socketId, username, url }
- `call` — emitted to competitors when a request is made. Payload: senderInfo
- `competitor_join` — emitted to the caller when a competitor joins the room. Payload: competitorInfo
- `sendBackCompetitors` — sent to competitors when `sendCompetitor` is used
- `receivePercentage` — sent to room participants to update progress
- `userDisconnect` — emitted when a user disconnects with the updated online users list

## Notes and caveats

- The server stores online users in-memory (`onlineUser` array). For production or multi-instance deployments, replace this with a shared store (Redis, database, etc.) and make room naming/state consistent across instances.
- CORS and Socket.IO allow connections from `CLIENT_URL`. Ensure that `CLIENT_URL` matches the client origin (for example `http://localhost:3000`).

## Troubleshooting

- If the client can't connect, verify the server is running and the `CLIENT_URL` value matches the client's origin.
- If you see `EADDRINUSE` errors, change `PORT` or stop the process that holds the port.

## Contributing

PRs welcome. If you add new socket events or endpoints, please update this README so the client and backend remain in sync.

## Author
APIPAWE KATOTO DANIEL
