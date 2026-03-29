# Realtime Chat - Multi-Room Chat Application

A production-ready multi-room chat application built with NestJS and React, featuring real-time messaging via WebSockets, JWT authentication, and PostgreSQL for data persistence.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture Overview](#architecture-overview)
3. [System Design](#system-design)
4. [Technology Stack](#technology-stack)
5. [Project Structure](#project-structure)
6. [Database Schema](#database-schema)
7. [API Design](#api-design)
8. [WebSocket Events](#websocket-events)
9. [Security Implementation](#security-implementation)
10. [Frontend Architecture](#frontend-architecture)
11. [Data Flow Diagrams](#data-flow-diagrams)

---

## Project Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           REALTIME CHAT APP                                 │
│                                                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────────────┐  │
│  │   Frontend      │    │   Backend       │    │      Database           │  │
│  │   (React)       │◄──►│   (NestJS)      │◄──►│    (PostgreSQL)         │  │
│  │                 │    │                 │    │                         │  │
│  │ • Auth UI       │    │ • REST API      │    │ • Users Table           │  │
│  │ • Chat Interface│    │ • WebSocket     │    │ • Rooms Table           │  │
│  │ • Room Browser  │    │ • JWT Auth      │    │ • Messages Table        │  │
│  └─────────────────┘    └─────────────────┘    └─────────────────────────┘  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Core Features

- **User Authentication**: Sign up, login with JWT access + refresh tokens
- **Room Management**: Create, join, leave chat rooms
- **Real-time Messaging**: Instant message delivery via WebSockets
- **Message Persistence**: All messages stored in PostgreSQL
- **User Presence**: Track online/offline status
- **Room History**: View previous messages when joining a room

---

## Architecture Overview

### High-Level Architecture

```
                                    ┌──────────────────────────────────────┐
                                    │           CLIENT LAYER               │
                                    │  ┌─────────────────────────────────┐ │
                                    │  │        React Application        │ │
                                    │  │  ┌─────────┐ ┌───────────────┐  │ │
                                    │  │  │ Zustand │ │ React Router  │  │ │
                                    │  │  │  Store  │ │ (Protected)   │  │ │
                                    │  │  └────┬────┘ └───────┬───────┘  │ │
                                    │  │       │              │          │ │
                                    │  │  ┌────▼──────────────▼──────┐   │ │
                                    │  │  │     Socket.io Client     │   │ │
                                    │  │  └───────────┬──────────────┘   │ │
                                    │  └──────────────│──────────────────┘ │
                                    └─────────────────│────────────────────┘
                                                      │
                            HTTP/REST ◄───────────────┼──────────────► WebSocket
                                                      │
                                    ┌─────────────────▼────────────────────┐
                                    │          API GATEWAY LAYER           │
                                    │                                      │
                                    │  ┌─────────────────────────────────┐ │
                                    │  │      NestJS Application         │ │
                                    │  │                                 │ │
                                    │  │  ┌─────────┐  ┌──────────────┐  │ │
                                    │  │  │ Guards  │  │ Interceptors │  │ │
                                    │  │  │ (Auth)  │  │ (Logging)    │  │ │
                                    │  │  └────┬────┘  └───────┬──────┘  │ │
                                    │  │       │               │         │ │
                                    │  └───────┼───────────────┼─────────┘ │
                                    │          │               │           │
                                    │  ┌───────▼───────────────▼───────┐   │
                                    │  │          Controllers          │   │
                                    │  │  ┌──────────┐ ┌──────────┐    │   │
                                    │  │  │ REST     │ │ WebSocket│    │   │
                                    │  │  │ (Auth,   │ │ (Chat    │    │   │
                                    │  │  │  Rooms)  │ │  Events) │    │   │
                                    │  │  └────┬─────┘ └─────┬────┘    │   │
                                    │  └───────┼─────────────┼─────────┘   │
                                    │          │             │             │
                                    │  ┌───────▼─────────────▼───────┐     │
                                    │  │          Services           │     │
                                    │  │  ┌────────┐ ┌────────────┐  │     │
                                    │  │  │ Auth   │ │ Chat       │  │     │
                                    │  │  │ Service│ │ Service    │  │     │
                                    │  │  └────────┘ └────┬───────┘  │     │
                                    │  └──────────────────┼──────────┘     │
                                    │                     │                │
                                    └─────────────────────┼────────────────┘
                                                          │
                                    ┌─────────────────────▼────────────────┐
                                    │              DATA LAYER              │
                                    │                                      │
                                    │  ┌─────────────────────────────────┐ │
                                    │  │        Prisma Client            │ │
                                    │  │                                 │ │
                                    │  │  ┌──────────┐ ┌───────────────┐ │ │
                                    │  │  │  User    │ │ Room          │ │ │
                                    │  │  │  Model   │ │ Model         │ │ │
                                    │  │  └──────────┘ └───────┬───────┘ │ │
                                    │  │                       │         │ │
                                    │  │  ┌────────────────────▼──────┐  │ │
                                    │  │  │      Message Model        │  │ │
                                    │  │  └───────────────────────────┘  │ │
                                    │  └─────────────────────────────────┘ │
                                    │                                      │
                                    │  ┌─────────────────────────────────┐ │
                                    │  │      PostgreSQL Database        │ │
                                    │  │                                 │ │
                                    │  │   users  │  rooms  │  messages  │ │
                                    │  │   (id)   │  (id)   │   (id)     │ │
                                    │  │  email   │  name   │  content   │ │
                                    │  │ password │ owner_id│  room_id   │ │
                                    │  │ username │         │ sender_id  │ │
                                    │  │          │         │   time     │ │
                                    │  └─────────────────────────────────┘ │
                                    └──────────────────────────────────────┘
```

### Request Flow

```
┌──────────┐         ┌──────────┐         ┌──────────┐         ┌──────────┐
│  Client  │         │  Guard   │         │Controller│         │ Service  │
└────┬─────┘         └────┬─────┘         └────┬─────┘         └────┬─────┘
     │                    │                    │                    │
     │  HTTP Request      │                    │                    │
     │ + JWT Token        │                    │                    │
     │──────────────────► │                    │                    │
     │                    │                    │                    │
     │              Verify JWT                 │                    │
     │                    │                    │                    │
     │            ┌───────┴───────┐            │                    │
     │            │  Valid?       │            │                    │
     │            │   No          │            │                    │
     │            └───────┬───────┘            │                    │
     │                    │                    │                    │
     │   401 Unauthorized │                    │                    │
     │◄───────────────────┘                    │                    │
     │                                         │                    │
     │           ┌───────┴───────┐             │                    │
     │           │  Valid?       │             │                    │
     │           │   Yes         │             │                    │
     │           └───────┬───────┘             │                    │
     │                   │                     │                    │
     │                   │   Continue          │                    │
     │                   │───────────────────► │                    │
     │                   │                     │                    │
     │                   │                     │  Process Request   │
     │                   │                     │───────────────────►
     │                   │                     │                    │
     │                   │                     │                    │  Business Logic
     │                   │                     │                    │  + DB Operations
     │                   │                     │                    │
     │                   │                     │   Response         │
     │                   │                     │◄───────────────────
     │                   │                     │                    │
     │  Response         │                     │                    │
     │◄──────────────────│                     │                    │
     │                   │                     │                    │
```

---

## System Design

### Authentication Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         AUTHENTICATION FLOW                                 │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────┐                              ┌─────────────────────────────┐
│   Client    │                              │         Backend             │
└──────┬──────┘                              └────────┬────────────────────┘
       │                                              │
       │  ┌─────────────────────────────────────┐     │
       │  │           SIGN UP FLOW              │     │
       │  └─────────────────────────────────────┘     │
       │                                              │
       │  POST /auth/signup { email, username, password }
       │─────────────────────────────────────────────►
       │                                              │
       │                                    ┌──────────▼──────────┐
       │                                    │   AuthController    │
       │                                    └──────────┬──────────┘
       │                                               │
       │                                    ┌──────────▼──────────┐
       │                                    │   AuthService       │
       │                                    │                     │
       │                                    │  1. Hash Password   │
       │                                    │  2. Create User     │
       │                                    │  3. Generate JWTs   │
       │                                    └──────────┬──────────┘
       │                                               │
       │                                    ┌──────────▼──────────┐
       │                                    │  Prisma Service     │
       │                                    │   (User Model)      │
       │                                    └───────────┬─────────┘
       │                                                │
       │                                                │ INSERT INTO users
       │                                                ▼
       │                                    ┌─────────────────────┐
       │                                    │    PostgreSQL       │
       │                                    │    users table      │
       │                                    └──────────┬──────────┘
       │                                               │
       │  { accessToken, refreshToken, user }          │
       │◄──────────────────────────────────────────────┘
       │
       │
       │  ┌─────────────────────────────────────┐      │
       │  │           LOGIN FLOW                │      │
       │  └─────────────────────────────────────┘      │
       │                                               │
       │  POST /auth/login { email, password }         │
       │─────────────────────────────────────────────►
       │                                               │
       │                                    ┌──────────▼──────────┐
       │                                    │   AuthController    │
       │                                    └──────────┬──────────┘
       │                                               │
       │                                    ┌──────────▼──────────┐
       │                                    │   AuthService       │
       │                                    │                     │
       │                                    │  1. Find User       │
       │                                    │  2. Verify Pass     │
       │                                    │  3. Generate JWTs   │
       │                                    └──────────┬──────────┘
       │                                               │
       │  { accessToken, refreshToken, user }          │
       │◄──────────────────────────────────────────────┘
       │                                               │   
       │                                               │
       │  ┌─────────────────────────────────────┐      │
       │  │         REFRESH TOKEN FLOW          │      │
       │  └─────────────────────────────────────┘      │
       │                                               │
       │  POST /auth/refresh { refreshToken }          │
       │──────────────────────────────────────────────►│
       │                                               │
       │                                    ┌──────────▼──────────┐
       │                                    │ JwtAuthGuard        │
       │                                    │ (Validates refresh) │
       │                                    └──────────┬──────────┘
       │                                               │
       │                                    ┌──────────▼──────────┐
       │                                    │   AuthService       │
       │                                    │                     │
       │                                    │  1. Verify Token    │
       │                                    │  2. Check Blacklist │
       │                                    │  3. Generate New    │
       │                                    │     Access Token    │
       │                                    └──────────┬──────────┘
       │                                               │
       │  { accessToken }                              │
       │◄──────────────────────────────────────────────┘
```

### WebSocket Connection Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    WEBSOCKET CONNECTION FLOW                                │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────┐                              ┌─────────────────────────────┐
│   Client    │                              │         Backend             │
└──────┬──────┘                              └──────────────┬──────────────┘
       │                                                    │
       │  1. Connect (with JWT in handshake auth)           │
       │───────────────────────────────────────────────────►
       │                                                    │
       │                                    ┌──────────▼──────────┐
       │                                    │    WsJwtGuard       │
       │                                    │                     │
       │                                    │ • Extract token     │
       │                                    │ • Verify JWT        │
       │                                    │ • Attach user       │
       │                                    └──────────┬──────────┘
       │                                               │
       │                                    ┌──────────▼──────────┐
       │                                    │    ChatGateway      │
       │                                    │                     │
       │                                    │ • OnConnect()       │
       │                                    │ • Join user to      │
       │                                    │   online users map  │
       │                                    │ • Emit 'online'     │
       │                                    │   to all clients    │
       │                                    └──────────┬──────────┘
       │                                               │
       │  Socket connected + socket.id                 │
       │◄──────────────────────────────────────────────┘
       │
       │  2. Subscribe to Room                         │
       │  { event: 'joinRoom', roomId: 'room_123' }    │
       │─────────────────────────────────────────────►
       │                                               │
       │                                    ┌──────────▼──────────┐
       │                                    │ @SubscribeMessage   │
       │                                    │  'joinRoom'         │
       │                                    │                     │
       │                                    │ • Verify room       │
       │                                    │   exists            │
       │                                    │ • Join socket to    │
       │                                    │   room namespace    │
       │                                    │ • Save to DB        │
       │                                    │ • Emit confirmation │
       │                                    └──────────┬──────────┘
       │                                               │
       │  { event: 'roomJoined', room: {...} }         │
       │◄─────────────────────────────────────────────
       │
       │  3. Send Message                              │
       │  { event: 'sendMessage', roomId, content }    │
       │─────────────────────────────────────────────►
       │                                               │
       │                                    ┌──────────▼──────────┐
       │                                    │ @SubscribeMessage   │
       │                                    │  'sendMessage'      │
       │                                    │                     │
       │                                    │ • Save to DB        │
       │                                    │ • Broadcast to      │
       │                                    │   room clients      │
       │                                    │ • Emit 'newMessage' │
       │                                    └──────────┬──────────┘
       │                                               │
       │  { event: 'newMessage', message: {...} }      │
       │◄──────────────────────────────────────────────│
       │                                               │
       │  (Same message broadcasted to all             │
       │   clients in the room)                        │
       │                                               │
       │  4. Leave Room                                │
       │  { event: 'leaveRoom', roomId }               │
       │──────────────────────────────────────────────►│
       │                                               │
       │                                    ┌──────────▼──────────┐
       │                                    │ @SubscribeMessage   │
       │                                    │  'leaveRoom'        │
       │                                    │                     │
       │                                    │ • Remove from       │
       │                                    │   socket room       │
       │                                    │ • Emit 'left'       │
       │                                    │ • Update online     │
       │                                    │   status            │
       │                                    └──────────┬──────────┘
       │                                               │
       │  5. Disconnect                                │
       │  (on browser close / refresh)                 │
       │──────────────────────────────────────────────►│
       │                                               │
       │                                    ┌──────────▼──────────┐
       │                                    │ handleDisconnect()  │
       │                                    │                     │
       │                                    │ • Remove from       │
       │                                    │   online users      │
       │                                    │ • Emit 'offline'    │
       │                                    │   to all clients    │
       │                                    └─────────────────────┘
```

---

## Technology Stack

### Backend Dependencies

```json
{
  "dependencies": {
    "@nestjs/common": "^11.0.1",
    "@nestjs/core": "^11.0.1",
    "@nestjs/platform-express": "^11.0.1",
    "@nestjs/platform-socket.io": "^11.0.1",
    "@nestjs/websockets": "^11.0.1",
    "@nestjs/passport": "^11.0.1",
    "@nestjs/jwt": "^11.0.1",
    "@prisma/client": "^5.x.x",
    "pg": "^8.x.x",
    "passport": "^0.7.x",
    "passport-jwt": "^4.x.x",
    "passport-local": "^1.x.x",
    "bcrypt": "^5.x.x",
    "class-validator": "^0.14.x",
    "class-transformer": "^0.5.x",
    "rxjs": "^7.8.1",
    "reflect-metadata": "^0.2.2"
  },
  "devDependencies": {
    "prisma": "^5.x.x"
  }
}
```

### Technology Roles

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         TECHNOLOGY STACK                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  FRAMEWORK & RUNTIME                                                        │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  NestJS 11          │  Node.js  │  TypeScript 5.x                    │   │
│  │  • Modular arch     │  • V8     │  • Strict typing                   │   │
│  │  • Dependency inj   │  • Events │  • Decorators                      │   │
│  │  • Decorators       │  • Async  │  • Interfaces                      │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  AUTHENTICATION                                                             │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  JWT (Access + Refresh)         │  Passport.js                        │  │
│  │  ┌────────────────────────┐     │  ┌────────────────────────────┐     │  │
│  │  │ Access Token           │     │  │ JWT Strategy               │     │  │
│  │  │ • Short-lived (15m)    │     │  │ • Verify token signature   │     │  │
│  │  │ • Contains user ID     │     │  │ • Extract payload          │     │  │
│  │  │ • For API auth         │     │  │ • Attach user to request   │     │  │
│  │  └────────────────────────┘     │  └────────────────────────────┘     │  │
│  │  ┌────────────────────────┐     │  ┌────────────────────────────┐     │  │
│  │  │ Refresh Token          │     │  │ Local Strategy             │     │  │
│  │  │ • Long-lived (7d)      │     │  │ • Verify credentials       │     │  │
│  │  │ • Stored in HTTP-only  │     │  │ • Used for login           │     │  │
│  │  │ • Can be blacklisted   │     │  └────────────────────────────┘     │  │
│  │  └────────────────────────┘     │                                     │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  REAL-TIME COMMUNICATION                                                    │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  Socket.io                    │  @nestjs/websockets                  │   │
│  │  ┌────────────────────────┐   │  ┌────────────────────────────────┐  │   │
│  │  │ WebSocket              │   │  │ Gateway (ChatGateway)          │  │   │
│  │  │ • Persistent conn      │   │  │ • @WebSocketGateway()          │  │   │
│  │  │ • Bi-directional       │   │  │ • @SubscribeMessage()          │  │   │
│  │  │ • Auto-reconnect       │   │  │ • @OnConnect()                 │  │   │
│  │  │ • Fallback to polling  │   │  │ • @OnDisconnect()              │  │   │
│  │  └────────────────────────┘   │  └────────────────────────────────┘  │   │
│  │  ┌────────────────────────┐   │  ┌────────────────────────────────┐  │   │
│  │  │ Namespaces & Rooms     │   │  │ Guards (WsJwtGuard)            │  │   │
│  │  │ • /chat namespace      │   │  │ • Authenticate WS connections  │  │   │
│  │  │ • Room per chat room   │   │  │ • Extract user from token      │  │   │
│  │  │ • Join/leave rooms     │   │  └────────────────────────────────┘  │   │
│  │  └────────────────────────┘   │                                      │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  DATABASE                                                                   │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  PostgreSQL                   │  Prisma                              │   │
│  │  ┌────────────────────────┐   │  ┌────────────────────────────────┐  │   │
│  │  │ • ACID compliance      │   │  │ Prisma Schema & Client         │  │   │
│  │  │ • Relations            │   │  │ • type-safe queries            │  │   │
│  │  │ • JSON support         │   │  │ • auto-generated types         │  │   │
│  │  │ • Full-text search     │   │  │ • migrations                   │  │   │
│  │  │ • Triggers/procedures  │   │  │ • relation resolvers           │  │   │
│  │  └────────────────────────┘   │  └────────────────────────────────┘  │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  VALIDATION                                                                 │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │  class-validator                 │  class-transformer                │   │
│  │  ┌───────────────────────────┐   │  ┌─────────────────────────────┐  │   │
│  │  │ @IsEmail(), @IsString()   │   │  │ Transform plain objects     │  │   │
│  │  │ @MinLength(), @MaxLength  │   │  │ into class instances        │  │   │
│  │  │ @IsNotEmpty(), @Matches   │   │  │ for validation              │  │   │
│  │  └───────────────────────────┘   │  └─────────────────────────────┘  │   │
│  └──────────────────────────────────────────────────────────────────────┘   │ 
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Project Structure

### Backend Structure (NestJS)

```
backend/
├── src/
│   ├── main.ts                      # Application bootstrap
│   ├── app.module.ts                 # Root module
│   │
│   ├── common/                      # Shared utilities
│   │   ├── decorators/
│   │   │   ├── current-user.decorator.ts
│   │   │   └── public.decorator.ts
│   │   ├── guards/
│   │   │   ├── jwt-auth.guard.ts
│   │   │   ├── ws-jwt.guard.ts
│   │   │   └── refresh-token.guard.ts
│   │   ├── interceptors/
│   │   │   └── logging.interceptor.ts
│   │   └── filters/
│   │       └── http-exception.filter.ts
│   │
│   ├── config/                      # Configuration
│   │   ├── database.config.ts
│   │   ├── jwt.config.ts
│   │   └── env.validation.ts
│   │
│   ├── prisma/                      # Prisma
│   │   ├── schema.prisma           # Database schema
│   │   └── migrations/             # Database migrations
│   │
│   ├── auth/                        # Authentication module
│   │   ├── auth.module.ts
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   ├── strategies/
│   │   │   ├── jwt.strategy.ts
│   │   │   └── local.strategy.ts
│   │   ├── dto/
│   │   │   ├── sign-up.dto.ts
│   │   │   ├── sign-in.dto.ts
│   │   │   └── refresh-token.dto.ts
│   │   └── interfaces/
│   │       ├── jwt-payload.interface.ts
│   │       └── tokens.interface.ts
│   │
│   ├── users/                       # Users module
│   │   ├── users.module.ts
│   │   ├── users.service.ts
│   │   ├── users.controller.ts
│   │   └── dto/
│   │       └── update-user.dto.ts
│   │
│   ├── rooms/                       # Rooms module
│   │   ├── rooms.module.ts
│   │   ├── rooms.service.ts
│   │   ├── rooms.controller.ts
│   │   └── dto/
│   │       ├── create-room.dto.ts
│   │       └── join-room.dto.ts
│   │
│   ├── messages/                    # Messages module
│   │   ├── messages.module.ts
│   │   ├── messages.service.ts
│   │   ├── messages.controller.ts
│   │   └── dto/
│   │       └── create-message.dto.ts
│   │
│   └── chat/                        # WebSocket gateway
│       ├── chat.module.ts
│       ├── chat.gateway.ts
│       └── chat.service.ts
│
├── test/
│   ├── app.e2e-spec.ts
│   └── jest-e2e.json
│
├── docker-compose.yml               # PostgreSQL setup
├── .env                             # Environment variables
├── .env.example                     # Example env file
├── package.json
├── tsconfig.json
├── nest-cli.json
└── README.md
```

### Frontend Structure (React)

```
frontend/
├── src/
│   ├── main.tsx                     # App bootstrap
│   ├── App.tsx                      # Root component
│   │
│   ├── api/                         # API clients
│   │   ├── axios.ts                 # Axios instance
│   │   ├── auth-api.ts
│   │   ├── rooms-api.ts
│   │   └── messages-api.ts
│   │
│   ├── components/                  # Reusable components
│   │   ├── ui/
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Card.tsx
│   │   │   └── Avatar.tsx
│   │   ├── chat/
│   │   │   ├── ChatWindow.tsx
│   │   │   ├── MessageList.tsx
│   │   │   ├── MessageInput.tsx
│   │   │   ├── RoomList.tsx
│   │   │   └── UserList.tsx
│   │   └── layout/
│   │       ├── Navbar.tsx
│   │       └── Sidebar.tsx
│   │
│   ├── pages/                       # Route pages
│   │   ├── auth/
│   │   │   ├── LoginPage.tsx
│   │   │   └── SignupPage.tsx
│   │   ├── chat/
│   │   │   ├── ChatPage.tsx
│   │   │   └── RoomPage.tsx
│   │   └── HomePage.tsx
│   │
│   ├── hooks/                       # Custom hooks
│   │   ├── useAuth.ts
│   │   ├── useSocket.ts
│   │   ├── useMessages.ts
│   │   └── useRooms.ts
│   │
│   ├── stores/                      # Zustand stores
│   │   ├── authStore.ts
│   │   └── chatStore.ts
│   │
│   ├── services/                    # Business logic
│   │   └── socket.ts                # Socket.io setup
│   │
│   ├── types/                       # TypeScript types
│   │   ├── user.ts
│   │   ├── room.ts
│   │   ├── message.ts
│   │   └── socket.ts
│   │
│   └── utils/                       # Utilities
│       ├── constants.ts
│       └── helpers.ts
│
├── public/
├── package.json
├── tsconfig.json
├── vite.config.ts
└── index.html
```

---

## Database Schema

### Entity Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       DATABASE SCHEMA                                       │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────┐         ┌─────────────────────────────┐
│           users             │         │           rooms             │
├─────────────────────────────┤         ├─────────────────────────────┤
│ id                          │         │ id                          │
│ uuid        PK              │         │ uuid        PK              │
│                             │         │                             │
│ username                    │         │ name                        │
│ email         UNIQUE        │         │ description                 │
│ password                    │         │ is_private                  │
│ display_name                │         │ created_at                  │
│ avatar_url                  │         │ updated_at                  │
│ created_at                  │         │                             │
│ updated_at                  │         │                             │
│ refresh_token_hash          │         │                             │
│ is_active                   │         │                             │
└─────────────────────────────┘         └─────────────────────────────┘
       ▲                                        ▲
       │                                        │
       │ 1:N                                    │ 1:N
       │                                        │
       │                                        │
       │                                        │
┌──────┴────────────────────────────────────────┴─────────────────────────┐
│                         messages                                        │
├─────────────────────────────────────────────────────────────────────────┤
│ id                                                                   PK │
│ uuid                                                                    │
│                                                                         │
│ content                                                                 │
│ sender_id    FK ──────────────────────► users.id                        │
│ room_id     FK ──────────────────────► rooms.id                         │
│                                                                         │
│ created_at                                                              │
│ is_edited                                                               │
│ is_deleted                                                              │
└─────────────────────────────────────────────────────────────────────────┘

                        RELATIONSHIP SUMMARY
                        ====================

┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│    users     │         │    rooms     │         │   messages   │
└──────┬───────┘         └───────┬──────┘         └──────────────┘
       │                         │                          ▲
       │                         │                          │
       │                         │                          │
       │ 1:N owns                │ 1:N has                  │ N:1 sent
       │                         │                          │
       ▼                         ▼                          │
       │                         │                          │
       │    ┌────────────────────┴────────────────────┐     │
       │    │                                         │     │
       │    │    M:N via Prisma implicit table        │     │
       │    │    (user <-> rooms via members)         │     │
       │    │    Creates _RoomMembers join table      │     │
       │    └─────────────────────────────────────────┘     │
       │                                                    │
       └────────────────────────────────────────────────────┘
```

### Prisma Many-to-Many Relations

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    IMPLICIT vs EXPLICIT RELATIONS                           │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                           IMPLICIT (Default)                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  model User {                                                               │
│    rooms  Room[]  @relation("RoomMembers")                                  │
│  }                                                                          │
│                                                                             │
│  model Room {                                                               │
│    members User[]  @relation("RoomMembers")                                 │
│  }                                                                          │
│                                                                             │
│  Result: Creates automatic join table "_RoomMembers"                        │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                             EXPLICIT (With metadata)                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  model User {                                                               │
│    id         String    @id @default(uuid())                                │
│    rooms      UserRoom[]                                                    │
│  }                                                                          │
│                                                                             │
│  model UserRoom {                                                           │
│    userId    String                                                         │
│    roomId    String                                                         │
│    joinedAt  DateTime @default(now())                                       │
│    role      String   @default("member")                                    │
│                                                                             │
│    user      User    @relation(fields: [userId], references: [id])          │
│    room      Room    @relation(fields: [roomId], references: [id])          │
│                                                                             │
│    @@id([userId, roomId])                                                   │
│  }                                                                          │
│                                                                             │
│  model Room {                                                               │
│    id      String    @id @default(uuid())                                   │
│    members UserRoom[]                                                       │
│  }                                                                          │
│                                                                             │
│  Result: Explicit join table with additional metadata                       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Prisma Schema Definition

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PRISMA SCHEMA                                       │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                           schema.prisma                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  generator client {                                                         │
│    provider = "prisma-client-js"                                            │
│  }                                                                          │
│                                                                             │
│  datasource db {                                                            │
│    provider = "postgresql"                                                  │
│    url      = env("DATABASE_URL")                                           │
│  }                                                                          │
│                                                                             │
│  model User {                                                               │
│    id            String    @id @default(uuid())                             │
│    email         String   @unique                                           │
│    username      String                                                     │
│    password      String                                                     │
│    displayName   String?                                                    │
│    avatarUrl     String?                                                    │
│    refreshToken  String?                                                    │
│    createdAt     DateTime  @default(now())                                  │
│    updatedAt     DateTime  @updatedAt                                       │
│                                                                             │
│    // Relations                                                             │
│    ownedRooms    Room[]    @relation("RoomOwner")                           │
│    messages      Message[]                                                  │
│    rooms         Room[]    @relation("RoomMembers")                         │
│                                                                             │
│    @@map("users")                                                           │
│  }                                                                          │
│                                                                             │
│  model Room {                                                               │
│    id          String    @id @default(uuid())                               │
│    name        String                                                       │
│    description String?                                                      │
│    isPrivate   Boolean   @default(false)                                    │
│    inviteCode  String?   @unique                                            │
│    createdAt   DateTime  @default(now())                                    │
│    updatedAt   DateTime  @updatedAt                                         │
│                                                                             │
│    // Relations                                                             │
│    ownerId     String                                                       │
│    owner      User      @relation("RoomOwner", fields: [ownerId],           │
│                                 references: [id], onDelete: Cascade)        │
│    messages    Message[]                                                    │
│    members     User[]    @relation("RoomMembers")                           │
│                                                                             │
│    @@map("rooms")                                                           │
│  }                                                                          │
│                                                                             │
│  model Message {                                                            │
│    id        String   @id @default(uuid())                                  │
│    content   String                                                         │
│    createdAt DateTime @default(now())                                       │
│    isEdited  Boolean  @default(false)                                       │
│    isDeleted Boolean  @default(false)                                       │
│                                                                             │
│    // Relations                                                             │
│    senderId  String                                                         │
│    sender    User     @relation(fields: [senderId], references: [id],       │
│                            onDelete: Cascade)                               │
│    roomId    String                                                         │
│    room      Room     @relation(fields: [roomId], references: [id],         │
│                          onDelete: Cascade)                                 │
│                                                                             │
│    @@index([roomId, createdAt])                                             │
│    @@map("messages")                                                        │
│  }                                                                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Prisma Client Usage

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PRISMA IN SERVICES                                  │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                           UsersService                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  import { Injectable } from '@nestjs/common';                               │
│  import { PrismaService } from '../prisma/prisma.service';                  │
│                                                                             │
│  @Injectable()                                                              │
│  export class UsersService {                                                │
│    constructor(private prisma: PrismaService) {}                            │
│                                                                             │
│    async findById(id: string) {                                             │
│      return this.prisma.user.findUnique({                                   │
│        where: { id },                                                       │
│      });                                                                    │
│    }                                                                        │
│                                                                             │
│    async findByEmail(email: string) {                                       │
│      return this.prisma.user.findUnique({                                   │
│        where: { email },                                                    │
│      });                                                                    │
│    }                                                                        │
│                                                                             │
│    async create(data: CreateUserDto) {                                      │
│      return this.prisma.user.create({                                       │
│        data: {                                                              │
│          ...data,                                                           │
│          password: await hashPassword(data.password),                       │
│        },                                                                   │
│      });                                                                    │
│    }                                                                        │
│                                                                             │
│    async updateRefreshToken(userId: string, token: string | null) {         │
│      return this.prisma.user.update({                                       │
│        where: { id: userId },                                               │
│        data: { refreshToken: token },                                       │
│      });                                                                    │
│    }                                                                        │
│  }                                                                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                           RoomsService                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  @Injectable()                                                              │
│  export class RoomsService {                                                │
│    constructor(private prisma: PrismaService) {}                            │
│                                                                             │
│    async create(data: CreateRoomDto, ownerId: string) {                     │
│      return this.prisma.room.create({                                       │
│        data: {                                                              │
│          ...data,                                                           │
│          ownerId,                                                           │
│          members: {                                                         │
│            connect: { id: ownerId },                                        │
│          },                                                                 │
│        },                                                                   │
│        include: { members: true },                                          │
│      });                                                                    │
│    }                                                                        │
│                                                                             │
│    async findAll() {                                                        │
│      return this.prisma.room.findMany({                                     │
│        where: { isPrivate: false },                                         │
│        include: {                                                           │
│          members: { select: { id: true, username: true } },                 │
│          _count: { select: { messages: true } },                            │
│        },                                                                   │
│      });                                                                    │
│    }                                                                        │
│                                                                             │
│    async joinRoom(roomId: string, userId: string) {                         │
│      return this.prisma.room.update({                                       │
│        where: { id: roomId },                                               │ 
│        data: {                                                              │
│          members: { connect: { id: userId } },                              │
│        },                                                                   │
│      });                                                                    │
│    }                                                                        │ 
│                                                                             │
│    async leaveRoom(roomId: string, userId: string) {                        │
│      return this.prisma.room.update({                                       │
│        where: { id: roomId },                                               │
│        data: {                                                              │
│          members: { disconnect: { id: userId } },                           │
│        },                                                                   │
│      });                                                                    │
│    }                                                                        │
│  }                                                                          │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                         MessagesService                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  @Injectable()                                                              │
│  export class MessagesService {                                             │
│    constructor(private prisma: PrismaService) {}                            │
│                                                                             │
│    async create(data: CreateMessageDto, senderId: string) {                 │
│      return this.prisma.message.create({                                    │
│        data: {                                                              │
│          content: data.content,                                             │
│          roomId: data.roomId,                                               │
│          senderId,                                                          │
│        },                                                                   │
│        include: {                                                           │
│          sender: {                                                          │
│            select: { id: true, username: true, avatarUrl: true },           │
│          },                                                                 │
│        },                                                                   │
│      });                                                                    │
│    }                                                                        │
│                                                                             │
│    async findByRoom(roomId: string, limit = 50, cursor?: string) {          │
│      return this.prisma.message.findMany({                                  │
│        where: { roomId, isDeleted: false },                                 │
│        take: limit + 1,                                                     │
│        ...(cursor && {                                                      │
│          cursor: { id: cursor },                                            │
│          skip: 1,                                                           │
│        }),                                                                  │
│        orderBy: { createdAt: 'desc' },                                      │
│        include: {                                                           │
│          sender: {                                                          │
│            select: { id: true, username: true, avatarUrl: true },           │
│          },                                                                 │
│        },                                                                   │
│      });                                                                    │
│    }                                                                        │
│                                                                             │
│    async update(id: string, content: string) {                              │
│      return this.prisma.message.update({                                    │
│        where: { id },                                                       │
│        data: { content, isEdited: true },                                   │
│      });                                                                    │
│    }                                                                        │
│                                                                             │
│    async softDelete(id: string) {                                           │
│      return this.prisma.message.update({                                    │
│        where: { id },                                                       │
│        data: { isDeleted: true },                                           │
│      });                                                                    │
│    }                                                                        │ 
│  }                                                                          │ 
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## API Design

### REST Endpoints

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            API ENDPOINTS                                    │
└─────────────────────────────────────────────────────────────────────────────┘

BASE URL: http://localhost:3000/api

═══════════════════════════════════════════════════════════════════════════════
                              AUTHENTICATION
═══════════════════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────────────────┐
│ POST /auth/signup                                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│ Create a new user account                                                   │
│                                                                             │
│ Request Body:                                                               │
│ {                                                                           │
│   "email": "user@example.com",                                              │
│   "username": "johndoe",                                                    │
│   "password": "SecurePass123!"                                              │
│ }                                                                           │
│                                                                             │
│ Response (201):                                                             │
│ {                                                                           │
│   "user": {                                                                 │
│     "id": "uuid",                                                           │
│     "email": "user@example.com",                                            │
│     "username": "johndoe",                                                  │
│     "displayName": null                                                     │
│   },                                                                        │
│   "accessToken": "eyJ...",                                                  │
│   "refreshToken": "eyJ..."                                                  │
│ }                                                                           │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ POST /auth/login                                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│ Authenticate user and get tokens                                            │
│                                                                             │
│ Request Body:                                                               │
│ {                                                                           │
│   "email": "user@example.com",                                              │
│   "password": "SecurePass123!"                                              │
│ }                                                                           │
│                                                                             │
│ Response (200):                                                             │
│ {                                                                           │
│   "user": { ... },                                                          │
│   "accessToken": "eyJ...",                                                  │
│   "refreshToken": "eyJ..."                                                  │
│ }                                                                           │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ POST /auth/refresh                                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│ Get new access token using refresh token                                    │
│                                                                             │
│ Request Body:                                                               │
│ {                                                                           │
│   "refreshToken": "eyJ..."                                                  │
│ }                                                                           │
│                                                                             │
│ Response (200):                                                             │
│ {                                                                           │
│   "accessToken": "eyJ..."                                                   │
│ }                                                                           │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ POST /auth/logout                                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│ Invalidate refresh token (optional)                                         │
│                                                                             │
│ Request Body:                                                               │
│ {                                                                           │
│   "refreshToken": "eyJ..."                                                  │
│ }                                                                           │
│                                                                             │
│ Response (200):                                                             │
│ {                                                                           │
│   "message": "Logged out successfully"                                      │
│ }                                                                           │
└─────────────────────────────────────────────────────────────────────────────┘

═══════════════════════════════════════════════════════════════════════════════
                                  USERS
═══════════════════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────────────────┐
│ GET /users/me                                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│ Get current user profile (Authenticated)                                    │
│                                                                             │
│ Response (200):                                                             │
│ {                                                                           │
│   "id": "uuid",                                                             │
│   "email": "user@example.com",                                              │
│   "username": "johndoe",                                                    │
│   "displayName": "John Doe",                                                │
│   "avatarUrl": "https://...",                                               │
│   "createdAt": "2024-01-01T00:00:00Z"                                       │
│ }                                                                           │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ PATCH /users/me                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│ Update current user profile (Authenticated)                                 │
│                                                                             │
│ Request Body:                                                               │
│ {                                                                           │
│   "displayName": "John Doe",                                                │
│   "avatarUrl": "https://..."                                                │
│ }                                                                           │
│                                                                             │
│ Response (200): User updated                                                │
└─────────────────────────────────────────────────────────────────────────────┘

═══════════════════════════════════════════════════════════════════════════════
                                  ROOMS
═══════════════════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────────────────┐
│ GET /rooms                                                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│ Get all public rooms (Authenticated)                                        │
│                                                                             │
│ Response (200):                                                             │
│ {                                                                           │
│   "rooms": [                                                                │
│     {                                                                       │
│       "id": "uuid",                                                         │
│       "name": "General",                                                    │
│       "description": "General chat room",                                   │
│       "isPrivate": false,                                                   │
│       "memberCount": 42,                                                    │
│       "createdAt": "2024-01-01T00:00:00Z"                                   │
│     }                                                                       │
│   ]                                                                         │
│ }                                                                           │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ POST /rooms                                                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│ Create a new room (Authenticated)                                           │
│                                                                             │
│ Request Body:                                                               │
│ {                                                                           │
│   "name": "Tech Talk",                                                      │
│   "description": "Discuss technology",                                      │
│   "isPrivate": false                                                        │
│ }                                                                           │
│                                                                             │
│ Response (201):                                                             │
│ {                                                                           │
│   "room": {                                                                 │
│     "id": "uuid",                                                           │
│     "name": "Tech Talk",                                                    │
│     ...                                                                     │
│   }                                                                         │
│ }                                                                           │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ GET /rooms/:id                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│ Get room details (Authenticated)                                            │
│                                                                             │
│ Response (200):                                                             │
│ {                                                                           │
│   "room": {                                                                 │
│     "id": "uuid",                                                           │
│     "name": "Tech Talk",                                                    │
│     "members": [...],                                                       │
│     "createdAt": "..."                                                      │
│   }                                                                         │
│ }                                                                           │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ POST /rooms/:id/join                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│ Join a room (Authenticated)                                                 │
│                                                                             │
│ Response (200):                                                             │
│ {                                                                           │
│   "message": "Joined room successfully",                                    │
│   "room": { ... }                                                           │
│ }                                                                           │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ POST /rooms/:id/leave                                                       │
├─────────────────────────────────────────────────────────────────────────────┤
│ Leave a room (Authenticated)                                                │
│                                                                             │
│ Response (200):                                                             │
│ {                                                                           │
│   "message": "Left room successfully"                                       │
│ }                                                                           │
└─────────────────────────────────────────────────────────────────────────────┘

═══════════════════════════════════════════════════════════════════════════════
                                MESSAGES
═══════════════════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────────────────┐
│ GET /rooms/:id/messages                                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│ Get message history for a room (Authenticated)                              │
│                                                                             │
│ Query Parameters:                                                           │
│   - limit: number (default: 50, max: 100)                                   │
│   - before: ISO date string (pagination)                                    │
│                                                                             │
│ Response (200):                                                             │
│ {                                                                           │
│   "messages": [                                                             │
│     {                                                                       │
│       "id": "uuid",                                                         │
│       "content": "Hello!",                                                  │
│       "sender": { "id": "uuid", "username": "johndoe" },                    │
│       "createdAt": "2024-01-01T00:00:00Z",                                  │
│       "isEdited": false                                                     │
│     }                                                                       │
│   ],                                                                        │
│   "hasMore": true                                                           │
│ }                                                                           │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## WebSocket Events

### Socket.io Event Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         WEBSOCKET EVENTS                                    │
└─────────────────────────────────────────────────────────────────────────────┘

═══════════════════════════════════════════════════════════════════════════════
                              CLIENT → SERVER
═══════════════════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────────────────┐
│ EVENT: joinRoom                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│ Description: User joins a chat room                                         │
│                                                                             │
│ Payload:                                                                    │
│ {                                                                           │
│   "roomId": "uuid"                                                          │
│ }                                                                           │
│                                                                             │
│ Server Response:                                                            │
│   • Emits 'roomJoined' to requesting client                                 │
│   • Emits 'userJoined' to all room members                                  │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ EVENT: leaveRoom                                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│ Description: User leaves a chat room                                        │
│                                                                             │
│ Payload:                                                                    │
│ {                                                                           │
│   "roomId": "uuid"                                                          │
│ }                                                                           │
│                                                                             │
│ Server Response:                                                            │
│   • Emits 'roomLeft' to requesting client                                   │
│   • Emits 'userLeft' to all room members                                    │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ EVENT: sendMessage                                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│ Description: Send a message to a room                                       │
│                                                                             │
│ Payload:                                                                    │
│ {                                                                           │
│   "roomId": "uuid",                                                         │
│   "content": "Hello everyone!"                                              │
│ }                                                                           │
│                                                                             │
│ Server Response:                                                            │
│   • Saves message to database                                               │
│   • Emits 'newMessage' to ALL clients in the room (including sender)        │
│   • Payload: { id, content, sender, roomId, createdAt }                     │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ EVENT: editMessage                                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│ Description: Edit an existing message                                       │
│                                                                             │
│ Payload:                                                                    │
│ {                                                                           │
│   "messageId": "uuid",                                                      │
│   "content": "Updated message content"                                      │
│ }                                                                           │
│                                                                             │
│ Server Response:                                                            │
│   • Verifies ownership                                                      │
│   • Updates message in database                                             │
│   • Emits 'messageEdited' to all room members                               │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ EVENT: deleteMessage                                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│ Description: Delete a message                                               │
│                                                                             │
│ Payload:                                                                    │
│ {                                                                           │
│   "messageId": "uuid"                                                       │
│ }                                                                           │
│                                                                             │
│ Server Response:                                                            │
│   • Verifies ownership or room ownership                                    │
│   • Marks message as deleted                                                │
│   • Emits 'messageDeleted' to all room members                              │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ EVENT: typing                                                               │
├─────────────────────────────────────────────────────────────────────────────┤
│ Description: Notify room members that user is typing                        │
│                                                                             │
│ Payload:                                                                    │
│ {                                                                           │
│   "roomId": "uuid",                                                         │
│   "isTyping": true                                                          │
│ }                                                                           │
│                                                                             │
│ Server Response:                                                            │
│   • Emits 'userTyping' to other room members (not sender)                   │
└─────────────────────────────────────────────────────────────────────────────┘

═══════════════════════════════════════════════════════════════════════════════
                              SERVER → CLIENT
═══════════════════════════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────────────────────┐
│ EVENT: connected                                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│ Sent immediately upon successful WebSocket connection                       │
│                                                                             │
│ Payload:                                                                    │
│ {                                                                           │
│   "socketId": "abc123",                                                     │
│   "userId": "uuid",                                                         │
│   "onlineUsers": [...]                                                      │
│ }                                                                           │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ EVENT: newMessage                                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│ Broadcast when a new message is sent                                        │
│                                                                             │
│ Payload:                                                                    │
│ {                                                                           │
│   "id": "uuid",                                                             │
│   "content": "Hello everyone!",                                             │
│   "sender": {                                                               │
│     "id": "uuid",                                                           │
│     "username": "johndoe",                                                  │
│     "avatarUrl": "https://..."                                              │
│   },                                                                        │
│   "roomId": "uuid",                                                         │
│   "createdAt": "2024-01-01T00:00:00Z"                                       │
│ }                                                                           │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ EVENT: userJoined                                                           │
├─────────────────────────────────────────────────────────────────────────────┤
│ Broadcast when a user joins a room                                          │
│                                                                             │
│ Payload:                                                                    │
│ {                                                                           │
│   "user": {                                                                 │
│     "id": "uuid",                                                           │
│     "username": "jane"                                                      │
│   },                                                                        │
│   "roomId": "uuid",                                                         │
│   "memberCount": 5                                                          │
│ }                                                                           │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ EVENT: userLeft                                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│ Broadcast when a user leaves a room                                         │
│                                                                             │
│ Payload:                                                                    │
│ {                                                                           │
│   "userId": "uuid",                                                         │
│   "roomId": "uuid",                                                         │
│   "memberCount": 4                                                          │
│ }                                                                           │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ EVENT: userOnline / userOffline                                             │
├─────────────────────────────────────────────────────────────────────────────┤
│ Broadcast when a user's online status changes                               │
│                                                                             │
│ Payload:                                                                    │
│ {                                                                           │
│   "userId": "uuid",                                                         │
│   "username": "johndoe",                                                    │
│   "status": "online" | "offline"                                            │
│ }                                                                           │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│ EVENT: error                                                                │
├─────────────────────────────────────────────────────────────────────────────┤
│ Sent when an error occurs                                                   │
│                                                                             │
│ Payload:                                                                    │
│ {                                                                           │
│   "code": "ROOM_NOT_FOUND",                                                 │
│   "message": "The specified room does not exist"                            │
│ }                                                                           │
└─────────────────────────────────────────────────────────────────────────────┘
```

### WebSocket Message Flow Diagram

```
┌─────────┐         ┌─────────────────┐         ┌─────────────────────────┐
│ Client  │         │   ChatGateway   │         │      ChatService        │
└────┬────┘         └─────────┬───────┘         └────────────┬────────────┘
     │                        │                              │
     │  sendMessage()         │                              │
     │  { roomId, content }   │                              │
     │───────────────────────►                               │
     │                        │                              │
     │                        │  @SubscribeMessage           │
     │                        │  Process message             │
     │                        │─────────────────────────────►│
     │                        │                              │
     │                        │                              │ Create message
     │                        │                              │ in database
     │                        │                              │
     │                        │  Return saved message        │
     │                        │◄─────────────────────────────│
     │                        │                              │
     │                        │  server.to(roomId)           │
     │                        │  .emit('newMessage', msg)    │
     │                        │                              │
     │  newMessage event      │                              │
     │◄───────────────────────│                              │
     │                        │                              │
     │  All other clients     │                              │
     │  in same room receive  │                              │
     │  the message           │                              │
```

---

## Security Implementation

### Authentication Guards

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      SECURITY IMPLEMENTATION                                │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                        JWT AUTHENTICATION FLOW                              │
└─────────────────────────────────────────────────────────────────────────────┘

                                HTTP Request
                                       │
                                       ▼
                         ┌─────────────────────────┐
                         │   Request Interceptor   │
                         │                         │
                         │ • Extract Authorization │
                         │   header                │
                         │ • Parse Bearer token    │
                         │ • Attach to request     │
                         └───────────┬─────────────┘
                                     │
                                     ▼
                         ┌─────────────────────────┐
                         │      JwtAuthGuard       │
                         │                         │
                         │ • Validate token        │
                         │ • Check expiration      │
                         │ • Extract payload       │
                         │ • Attach user to req    │
                         └───────────┬─────────────┘
                                     │
                          ┌──────────┴──────────┐
                          │                     │
                     Valid                  Invalid
                          │                     │
                          ▼                     ▼
                   Continue              401 Unauthorized
                          │                     │
                          ▼                     ▼
                   Controller           Error Response
                          │
                          ▼
                    Service Layer


┌─────────────────────────────────────────────────────────────────────────────┐
│                         PASSWORD SECURITY                                   │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                              Sign Up                                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  User Input: "MyPassword123!"                                               │
│                      │                                                      │
│                      ▼                                                      │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  Bcrypt Hashing (rounds: 12)                                        │    │
│  │                                                                     │    │
│  │  Input:     "MyPassword123!"                                        │    │
│  │  Salt:      "$2b$12$LQv3c1yqBWVHxkd0LHAkCO"                         │    │
│  │  Hash:      "$2b$12$LQv3c1yqBWVHxkd0LHAkCO.                         │    │
│  │                   9S4wF8V3O1R7L6K5J4I3H2G1F0E"                      │    │
│  │                                                                     │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                      │                                                      │
│                      ▼                                                      │
│  Database: Stored hash (never the actual password )                         │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                              Login                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  User Input: "MyPassword123!"  ─────────────────────────┐                   │
│                                                         │                   │
│                                                         ▼                   │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  Bcrypt Compare                                                     │    │
│  │                                                                     │    │
│  │  Input:     "MyPassword123!"                                        │    │
│  │  Stored:    "$2b$12$LQv3c1yqBWVHxkd0LHAkCO..."                      │    │
│  │                                                                     │    │
│  │  Result:    Match = true/false                                      │    │
│  │                                                                     │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                      │                                                      │
│           ┌──────────┴──────────┐                                           │
│        Match                No Match                                        │
│           │                       │                                         │
│           ▼                       ▼                                         │
│    Generate JWTs          401 Unauthorized                                  │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘


┌─────────────────────────────────────────────────────────────────────────────┐
│                     TOKEN SECURITY BEST PRACTICES                           │
└─────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│  ACCESS TOKEN                          │  REFRESH TOKEN                      │
├────────────────────────────────────────┼─────────────────────────────────────┤
│                                        │                                     │
│  Lifetime: 15 minutes                  │  Lifetime: 7 days                   │
│                                        │                                     │
│  Storage: Memory / React state         │  Storage: HTTP-only cookie          │
│  (NOT localStorage)                    │  (or secure localStorage)           │
│                                        │                                     │
│  Sent in: Authorization header         │  Sent in: Cookie or body            │
│  "Bearer eyJ..."                       │                                     │
│                                        │                                     │
│  Purpose: API authentication           │  Purpose: Get new access token      │
│                                        │                                     │
│  Rotation: On every refresh request    │  Rotation: On every refresh         │
│                                        │                                     │
└──────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                         INPUT VALIDATION                                    │
└─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  class-validator decorators:                                                │
│                                                                             │
│  ┌─────────────────────────┐  ┌─────────────────────────┐                   │
│  │ User SignUp DTO         │  │ Message DTO             │                   │
│  ├─────────────────────────┤  ├─────────────────────────┤                   │
│  │ @IsEmail()              │  │ @IsString()             │                   │
│  │ @IsNotEmpty()           │  │ @MinLength(1)           │                   │
│  │ @MinLength(3)           │  │ @MaxLength(2000)        │                   │
│  │ @MaxLength(20)          │  │ @Matches regex for      │                   │
│  │ @Matches(passwordRegex) │  │   profanity filter      │                   │
│  └─────────────────────────┘  └─────────────────────────┘                   │
│                                                                             │
│  Results:                                                                   │
│  • Reject invalid input BEFORE processing                                   │
│  • Return detailed error messages                                           │
│  • Prevent SQL injection                                                    │
│  • Prevent XSS attacks                                                      │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Frontend Architecture

### Component Hierarchy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        FRONTEND ARCHITECTUR                                 │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                        APPLICATION STRUCTURE                                │
└─────────────────────────────────────────────────────────────────────────────┘

                           ┌──────────────┐
                           │    App       │
                           │  (Router)    │
                           └──────┬───────┘
                                  │
              ┌───────────────────┼───────────────────┐
              │                   │                   │
              ▼                   ▼                   ▼
     ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
     │  LoginPage   │    │  SignupPage  │    │  ChatPage    │
     │              │    │              │    │              │
     │ (Public)     │    │ (Public)     │    │ (Protected)  │
     └──────────────┘    └──────────────┘    └──────┬───────┘
                                                    │
                         ┌───────────────────────────┼───────────────────────────┐
                         │                           │                           │
                         ▼                           ▼                           ▼
                ┌──────────────┐            ┌───────────────┐            ┌──────────────┐
                │   Sidebar    │            │   ChatArea    │            │   UserPanel  │
                │              │            │               │            │              │
                │ • RoomList   │            │ • MessageList │            │ • Profile    │
                │ • UserInfo   │            │ • MessageInput│            │ • Settings   │
                └──────┬───────┘            └──────┬────────┘            └──────────────┘
                       │                          │
                       │                          │
                       ▼                          ▼
                ┌──────────────┐            ┌──────────────┐
                │  RoomCard    │            │   Message    │
                │              │            │              │
                │ • Name       │            │ • Sender     │
                │ • Members    │            │ • Content    │
                │ • LastMsg    │            │ • Timestamp  │
                └──────────────┘            └──────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                      STATE MANAGEMENT (ZUSTAND)                             │
└─────────────────────────────────────────────────────────────────────────────┘

                              ┌─────────────────┐
                              │   Zustand       │
                              │   Stores        │
                              └─────────┬───────┘
                                        │
              ┌─────────────────────────┼─────────────────────────┐
              │                         │                         │
              ▼                         ▼                         ▼
     ┌────────────────┐        ┌────────────────┐        ┌────────────────┐
     │  authStore     │        │   chatStore    │        │  socketStore   │
     ├────────────────┤        ├────────────────┤        ├────────────────┤
     │                │        │                │        │                │
     │ user: User     │        │ rooms: Room[]  │        │ connected: bool│
     │ isAuthenticated│        │ currentRoom    │        │ socket: Socket │
     │ accessToken    │        │ messages       │        │ onlineUsers    │
     │                │        │ typingUsers    │        │                │
     │ setUser()      │        │                │        │ connect()      │
     │ login()        │        │ setRooms()     │        │ disconnect()   │
     │ logout()       │        │ addMessage()   │        │ emit()         │
     │ refreshToken() │        │ joinRoom()     │        │                │
     │                │        │ leaveRoom()    │        │                │
     └────────────────┘        └────────────────┘        └────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                        PROTECTED ROUTE LOGIC                                │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                      ProtectedRoute Component                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                                                                      │   │
│  │   const ProtectedRoute = ({ children }) => {                         │   │
│  │                                                                      │   │
│  │     const { isAuthenticated, isLoading } = useAuthStore();           │   │
│  │     const navigate = useNavigate();                                  │   │
│  │                                                                      │   │
│  │     useEffect(() => {                                                │   │
│  │       if (!isLoading && !isAuthenticated) {                          │   │
│  │         navigate('/login');                                          │   │
│  │       }                                                              │   │
│  │     }, [isAuthenticated, isLoading, navigate]);                      │   │
│  │                                                                      │   │
│  │     if (isLoading) return <LoadingSpinner />;                        │   │
│  │                                                                      │   │
│  │     return isAuthenticated ? children : null;                        │   │
│  │   };                                                                 │   │
│  │                                                                      │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
│  Route Configuration:                                                       │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                                                                      │   │
│  │   <Routes>                                                           │   │
│  │     <Route path="/login" element={<LoginPage />} />                  │   │
│  │     <Route path="/signup" element={<SignupPage />} />                │   │
│  │                                                                      │   │
│  │     <Route                                                           │   │
│  │       path="/chat"                                                   │   │
│  │       element={                                                      │   │
│  │         <ProtectedRoute>                                             │   │
│  │           <ChatPage />                                               │   │
│  │         </ProtectedRoute>                                            │   │
│  │       }                                                              │   │
│  │     />                                                               │   │
│  │   </Routes>                                                          │   │
│  │                                                                      │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Socket.io Client Setup

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      SOCKET.IO CLIENT INTEGRATION                           │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                         Socket Service                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                                                                      │   │
│  │   // socket.ts                                                       │   │
│  │                                                                      │   │
│  │   import { io, Socket } from 'socket.io-client';                     │   │
│  │   import { store } from './stores/authStore';                        │   │
│  │                                                                      │   │
│  │   class SocketService {                                              │   │
│  │     private socket: Socket | null = null;                            │   │
│  │                                                                      │   │
│  │     connect() {                                                      │   │
│  │       const { accessToken } = store.getState();                      │   │
│  │                                                                      │   │
│  │       this.socket = io('http://localhost:3000', {                    │   │
│  │         auth: { token: accessToken },                                │   │
│  │         transports: ['websocket', 'polling'],                        │   │
│  │         reconnection: true,                                          │   │
│  │         reconnectionAttempts: 5,                                     │   │
│  │         reconnectionDelay: 1000,                                     │   │
│  │       });                                                            │   │
│  │                                                                      │   │
│  │       this.socket.on('connect', () => {                              │   │
│  │         console.log('Socket connected:', this.socket.id);            │   │
│  │       });                                                            │   │
│  │                                                                      │   │
│  │       this.socket.on('disconnect', () => {                           │   │
│  │         console.log('Socket disconnected');                          │   │
│  │       });                                                            │   │
│  │     }                                                                │   │
│  │                                                                      │   │
│  │     disconnect() {                                                   │   │
│  │       this.socket?.disconnect();                                     │   │
│  │       this.socket = null;                                            │   │
│  │     }                                                                │   │
│  │                                                                      │   │
│  │     emit(event: string, data: any) {                                 │   │   
│  │       this.socket?.emit(event, data);                                │   │
│  │     }                                                                │   │
│  │                                                                      │   │
│  │     on(event: string, callback: Function) {                          │   │
│  │       this.socket?.on(event, callback);                              │   │
│  │     }                                                                │   │
│  │                                                                      │   │
│  │     off(event: string) {                                             │   │
│  │       this.socket?.off(event);                                       │   │
│  │     }                                                                │   │
│  │   }                                                                  │   │
│  │                                                                      │   │
│  │   export const socketService = new SocketService();                  │   │
│  │                                                                      │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                         Hook: useSocket                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐   │
│  │                                                                      │   │
│  │   export const useSocket = () => {                                   │   │
│  │     const [connected, setConnected] = useState(false);               │   │
│  │     const socketRef = useRef<Socket | null>(null);                   │   │
│  │                                                                      │   │
│  │     useEffect(() => {                                                │   │
│  │       const { accessToken, isAuthenticated } = useAuthStore();       │   │
│  │                                                                      │   │
│  │       if (!isAuthenticated || !accessToken) return;                  │   │
│  │                                                                      │   │
│  │       // Initialize socket with token                                │   │
│  │       socketRef.current = io(SOCKET_URL, {                           │   │
│  │         auth: { token: accessToken },                                │   │
│  │       });                                                            │   │
│  │                                                                      │   │
│  │       socketRef.current.on('connect', () => {                        │   │
│  │         setConnected(true);                                          │   │
│  │       });                                                            │   │
│  │                                                                      │   │
│  │       return () => {                                                 │   │
│  │         socketRef.current?.disconnect();                             │   │
│  │       };                                                             │   │
│  │     }, []);                                                          │   │
│  │                                                                      │   │
│  │     return { socket: socketRef.current, connected };                 │   │
│  │   };                                                                 │   │
│  │                                                                      │   │
│  └──────────────────────────────────────────────────────────────────────┘   │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Data Flow Diagrams

### Complete Message Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      COMPLETE MESSAGE FLOW                                  │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│  User   │     │ React   │     │ Socket  │     │ NestJS  │     │   DB    │
│ (Send)  │     │ Client  │     │ Client  │     │ Gateway │     │         │
└────┬────┘     └────┬────┘     └────┬────┘     └────┬────┘     └────┬────┘
     │               │               │               │               │
     │ Type message  │               │               │               │
     │─────────────► │               │               │               │
     │               │               │               │               │
     │               │ Send to socket│               │               │
     │               │─────────────► │               │               │
     │               │               │               │               │
     │               │               │ Emit 'sendMessage'            │
     │               │               │─────────────► │               │
     │               │               │               │               │
     │               │               │               │ Validate      │
     │               │               │               │ + Save        │
     │               │               │               │─────────────► │
     │               │               │               │               │
     │               │               │               │ Confirm save  │
     │               │               │               │◄───────────── │
     │               │               │               │               │
     │               │               │ Broadcast to room             │
     │               │               │◄──────────────│               │
     │               │               │               │               │
     │               │ Receive       │               │               │
     │               │◄──────────────│               │               │
     │               │               │               │               │
     │               │ Update UI     │               │               │
     │ Display       │               │               │               │
     │◄──────────────│               │               │               │
     │               │               │               │               │ 
```

### Authentication Flow with Refresh Token

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                 AUTHENTICATION WITH REFRESH TOKEN FLOW                      │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│  User   │     │ React   │     │ Axios   │     │ NestJS  │     │   DB    │
│ Browser │     │ App     │     │ Instance│     │ Server  │     │         │
└────┬────┘     └────┬────┘     └────┬────┘     └────┬────┘     └────┬────┘
     │               │               │               │               │
     │ Login creds   │               │               │               │
     │────────────►  │               │               │               │
     │               │               │               │               │
     │               │ POST /auth/login              │               │
     │               │──────────────►│               │               │
     │               │               │               │               │
     │               │               │ POST /auth/login              │
     │               │               │───────────── ►│               │
     │               │               │               │               │
     │               │               │               │ Verify creds  │
     │               │               │               │──────────────►│
     │               │               │               │               │
     │               │               │               │ Get user      │
     │               │               │               │──────────────►│
     │               │               │               │               │
     │               │               │               │ Gen tokens    │
     │               │               │               │               │
     │               │ Tokens + User │               │               │
     │               │               │◄──────────────│               │
     │               │               │               │               │
     │               │ Store in memory + cookie      │               │
     │               │◄──────────────│               │               │
     │               │               │               │               │
     │ Logged in     │               │               │               │
     │◄──────────────│               │               │               │
     │               │               │               │               │
     │───────────────│───────────────│───────────────│───────────────│
     │               │               │               │               │
     │               │ API Request + Access Token    │               │
     │               │─────────────►│                │               │
     │               │              │                │               │
     │               │              │ Validate JWT   │               │
     │               │              │───────────────►│               │
     │               │              │                │               │
     │               │              │                │ Token OK?     │
     │               │              │                │────┬────      │
     │               │              │                │    │          │
     │               │ Response     │    ┌───────────▼───────┐       │
     │               │◄─────────────│    │ Token Expired?    │       │
     │               │              │    │     No            │       │
     │               │              │    └─────────┬─────────┘       │
     │               │              │              │                 │
     │               │ Response     │              │                 │
     │               │◄─────────────│              │                 │
     │               │              │              │                 │ 
     │───────────────│──────────────│──────────────│─────────────────│
     │               │              │              │                 │
     │               │              │              │ Token Expired   │
     │               │              │              │◄────────────    │
     │               │              │              │                 │
     │               │              │ 401 + Refresh Token            │
     │               │◄─────────────│              │                 │
     │               │              │              │                 │
     │               │ POST /auth/refresh          │                 │
     │               │─────────────►│              │                 │
     │               │              │              │                 │
     │               │              │ POST /auth/refresh             │
     │               │              │─────────────►│                 │
     │               │              │              │                 │
     │               │              │              │ Verify refresh  │
     │               │              │              │─────────────►   │
     │               │              │              │                 │
     │               │              │              │ New Access      │
     │               │              │              │─────────────►   │
     │               │              │              │                 │
     │               │ New Access Token            │                 │
     │               │◄─────────────│              │                 │
     │               │              │              │                 │
     │               │ Retry original request      │                 │
     │               │─────────────►│              │                 │
     │               │              │              │                 │
     │               │              │              │ Success         │
     │               │              │◄─────────────│                 │
     │               │              │              │                 │
     │               │◄─────────────│              │                 │
     │               │              │              │                 │
```

---

## Configuration

### Environment Variables

```bash
# Backend (.env)
NODE_ENV=development
PORT=3000

# Database
DATABASE_HOST=localhost
DATABASE_PORT=5432
DATABASE_USER=chat_user
DATABASE_PASSWORD=chat_pass
DATABASE_NAME=chat_db

# JWT
JWT_ACCESS_SECRET=your-access-secret-key-here
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_SECRET=your-refresh-secret-key-here
JWT_REFRESH_EXPIRES_IN=7d

# CORS
CORS_ORIGIN=http://localhost:5173
```

### Docker Compose for Database

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16-alpine
    container_name: chat_db
    environment:
      POSTGRES_USER: chat_user
      POSTGRES_PASSWORD: chat_pass
      POSTGRES_DB: chat_db
    ports:
      - '5432:5432'
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U chat_user']
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

---

## Getting Started

### Prerequisites

- Node.js 18+
- PostgreSQL 16+
- pnpm (recommended) or npm

### Installation

```bash
# Clone repository
git clone <repo-url>
cd realtime-chat

# Backend setup
cd backend
pnpm install

# Start PostgreSQL
docker-compose up -d

# Generate Prisma Client
pnpm prisma generate

# Run migrations
pnpm prisma migrate dev

# Push schema to database (alternative to migrations)
pnpm prisma db push

# Start development server
pnpm start:dev
```

---

## Learning Outcomes

This project covers:

1. **NestJS Fundamentals**
   - Modules, Controllers, Services
   - Dependency Injection
   - Decorators & Middleware

2. **Authentication & Security**
   - JWT tokens (access + refresh)
   - Password hashing with bcrypt
   - Guards & Interceptors
   - Input validation

3. **Real-time Communication**
   - WebSocket protocol
   - Socket.io integration
   - Room management
   - Event-driven architecture

4. **Database Design**

- Prisma schema & models
- PostgreSQL schema
- Prisma migrations
- Type-safe queries with Prisma Client

5. **Frontend Architecture**
   - React with TypeScript
   - Zustand state management
   - Protected routes
   - Socket.io client

---

## License

MIT
