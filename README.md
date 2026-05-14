# 🎭 Smart Mafia — AI-Powered Party Game

> A real-time multiplayer Mafia party game with an AI narrator powered by OpenAI GPT-4o-mini.
> Players are assigned secret roles, survive the night, vote out suspects, and listen to dramatic AI-generated narration for every game event.

![NestJS](https://img.shields.io/badge/NestJS-10-E0234E?style=flat-square&logo=nestjs)
![Next.js](https://img.shields.io/badge/Next.js-15-000000?style=flat-square&logo=nextdotjs)
![Socket.io](https://img.shields.io/badge/Socket.io-4-010101?style=flat-square&logo=socketdotio)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-336791?style=flat-square&logo=postgresql)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o--mini-412991?style=flat-square&logo=openai)

---

## ✨ Features

| Feature | Details |
|---------|---------|
| 🔐 **Auth** | JWT access + refresh tokens, bcrypt password hashing |
| 🎮 **Real-time game** | Socket.io gateway — roles, phases, votes, chat all sync live |
| 🤖 **AI narrator** | GPT-4o-mini generates dramatic narration for every game event |
| 🎭 **Roles** | Mafia, Detective, Doctor, Civilian — all with unique night actions |
| 🗳️ **Voting** | Day voting with automatic elimination on majority |
| 🌙 **Night actions** | Mafia kills, Detective investigates, Doctor protects — resolved server-side |
| 🏠 **Rooms** | Create / join rooms with 4-digit codes, avatar picker |
| 🐳 **Docker** | Full stack runs with a single `docker compose up` |

---

## 🏗️ Architecture

```
        ┌─────────────────────────────────────────────┐
        │   Next.js 15 (App Router)  ·  CSS Modules   │
        └──────────────────┬──────────────────────────┘
                           │  REST + Socket.io
          ┌────────────────┼────────────────┐
          │                │                │
   ┌──────▼──────┐  ┌──────▼──────┐  ┌─────▼──────┐
   │auth-service │  │game-service │  │ ai-service │
   │   :3001     │  │ :3002/:3012 │  │   :3003    │
   │             │  │             │  │            │
   │ JWT + Users │  │ Rooms, Game │  │ GPT-4o-m.  │
   │  bcrypt     │  │ Socket.io   │  │ Narration  │
   └──────┬──────┘  └──────┬──────┘  └────────────┘
          │                │
   ┌──────▼────────────────▼──────┐
   │       PostgreSQL :5432       │
   │  users · rooms · players     │
   │  game_states                 │
   └──────────────────────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| **Backend** | NestJS 10, TypeScript, Prisma ORM, JWT auth, bcrypt |
| **Real-time** | Socket.io 4 (WebSocket gateway in `game-service`) |
| **Frontend** | Next.js 15 (App Router), CSS Modules |
| **AI** | OpenAI GPT-4o-mini — generates narration and chat responses |
| **Database** | PostgreSQL 15 |
| **Infrastructure** | Docker Compose (all services + DB in one command) |

---

## 🚀 Getting Started

### Prerequisites

- Docker & Docker Compose
- Node.js 18+ *(only for local dev without Docker)*
- OpenAI API key *(optional — mock responses used without it)*

### 1. Clone and configure

```bash
git clone https://github.com/a1exmsq/smart-mafia-monorepo.git
cd smart-mafia-monorepo/backend
```

Copy example env files:

```bash
cp auth-service/.env.example auth-service/.env
cp game-service/.env.example game-service/.env
cp ai-service/.env.example ai-service/.env
```

Fill in `ai-service/.env`:

```env
OPENAI_API_KEY=sk-...    # get at platform.openai.com
```

### 2. Start everything with Docker

```bash
docker compose up -d
```

### 3. Run database migrations

```bash
docker compose exec auth-service npx prisma migrate deploy
docker compose exec game-service npx prisma migrate deploy
```

### 4. Run the frontend

```bash
cd ../frontend
npm install
npm run dev
```

App → `http://localhost:3000`

---

## 📡 API Endpoints

### Auth Service — `http://localhost:3001`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/auth/register` | Register a player |
| `POST` | `/auth/login` | Login → JWT tokens |
| `POST` | `/auth/refresh` | Refresh access token |
| `GET`  | `/users/me` | Current user profile |

### Game Service — `http://localhost:3002`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/rooms` | Create a room |
| `GET`  | `/rooms` | List active rooms |
| `PATCH`| `/rooms/:id/start` | Start game (host) |
| `POST` | `/players/join` | Join a room |
| `POST` | `/game/:roomId/init` | Initialize game state |
| `POST` | `/game/:roomId/advance` | Advance phase |
| `POST` | `/game/:roomId/vote` | Cast a vote |

### AI Service — `http://localhost:3003`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/ai/narrate` | Generate game narration |
| `POST` | `/ai/chat` | Chat with the AI host |

Swagger UI available at `/api/docs` on each service.

---

## 🔌 Socket.io Events

**Namespace:** `/game` · **Auth:** JWT token in `socket.auth`

| Client → Server | Description |
|----------------|-------------|
| `join_room` | Join a game room |
| `cast_vote` | Vote to eliminate a player |
| `request_ai_narration` | Request AI narration |
| `send_message` | Send a chat message |

| Server → Client | Description |
|----------------|-------------|
| `game_started` | Game begins, roles assigned |
| `your_role` | Player's private role (only to that player) |
| `phase_changed` | Day / Night phase transition |
| `player_eliminated` | A player has been eliminated |
| `ai_narration` | AI narrator text |
| `game_over` | Game result |

---

## 🗂️ Repository Structure

```
smart-mafia-monorepo/
├── backend/                  # NestJS microservices
│   ├── auth-service/         # JWT auth + users (:3001)
│   ├── game-service/         # Game logic + Socket.io (:3002/:3012)
│   ├── ai-service/           # OpenAI narrator (:3003)
│   └── docker-compose.yml
└── frontend/                 # Next.js 15 SPA (:3000)
    ├── app/                  # App Router pages
    └── components/
        └── GameRoom/         # Main game UI — roles, voting, chat, AI chat
```

---

## 💡 Technical Highlights

- **Microservice architecture** — three independent NestJS services with their own DB connections, communicating via HTTP and Socket.io
- **Real-time sync** — Socket.io gateway in `game-service` broadcasts every state change (phase, vote, elimination) to all room participants
- **Private role delivery** — `your_role` event is emitted only to the specific player's socket, so roles are never leaked to others
- **AI narration** — after each phase transition, the AI service receives the full game context and generates a suspenseful in-character narration
- **JWT microservice auth** — game-service verifies tokens independently (same `JWT_SECRET`) without calling auth-service at runtime

---

## 👤 Authors

This was a team academic project (vibe-coding session).

| Member | Role |
|--------|------|
| Tymofii Snisarenko | Scrum Master, QA, DevOps |
| Andrii Butenko | Frontend & UI/UX |
| Artem Kulinich | Fullstack, AI & API |
| **Aliaksandr Dailid** | Backend — Game Logic |

[![GitHub](https://img.shields.io/badge/GitHub-a1exmsq-181717?style=flat-square&logo=github)](https://github.com/a1exmsq)
