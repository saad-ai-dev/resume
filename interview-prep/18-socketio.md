# 18. SOCKET.IO — REAL-TIME COMMUNICATION

## 18.1 What is Socket.IO?
Socket.IO enables **real-time, bidirectional, event-based communication** between client and server. Built on WebSocket protocol with fallbacks.

## 18.2 Architecture

```
┌─────────┐    WebSocket     ┌─────────┐
│ Client 1 │ <──────────────> │         │
├─────────┤                   │ Socket  │
│ Client 2 │ <──────────────> │   .IO   │
├─────────┤                   │ Server  │
│ Client 3 │ <──────────────> │         │
└─────────┘                   └─────────┘
```

## 18.3 Implementation

```javascript
// Server
const io = require('socket.io')(server, {
  cors: { origin: '*' },
});

io.on('connection', (socket) => {
  console.log('Connected:', socket.id);

  // Join a room (e.g., match room, chat room)
  socket.on('joinRoom', (roomId) => {
    socket.join(roomId);
    socket.to(roomId).emit('userJoined', { userId: socket.userId });
  });

  // Send message to room
  socket.on('sendMessage', ({ roomId, message }) => {
    io.to(roomId).emit('newMessage', {
      from: socket.userId,
      message,
      timestamp: Date.now(),
    });
  });

  // Typing indicator
  socket.on('typing', (roomId) => {
    socket.to(roomId).emit('userTyping', socket.userId);
  });

  socket.on('disconnect', () => {
    console.log('Disconnected:', socket.id);
  });
});

// Emit to specific user
io.to(socketId).emit('notification', data);

// Emit to all users in a room
io.to(roomId).emit('event', data);

// Broadcast to all except sender
socket.broadcast.emit('event', data);
```

### Scaling Socket.IO with Redis Adapter
```javascript
const { createAdapter } = require('@socket.io/redis-adapter');
const { createClient } = require('redis');

const pubClient = createClient({ url: 'redis://localhost:6379' });
const subClient = pubClient.duplicate();

io.adapter(createAdapter(pubClient, subClient));
// Now Socket.IO works across multiple server instances
```

## 18.4 Your Real-World Usage

| Project | Socket.IO Use Case |
|---------|-------------------|
| **Zamulk** | Live chat, audio/video calls between buyers/sellers/agents, real-time bidding in auction room |
| **NowVPlay** | Instant match invitations, booking confirmations, live activity updates |
| **US Ride** | Real-time ride tracking, driver-rider matching |

