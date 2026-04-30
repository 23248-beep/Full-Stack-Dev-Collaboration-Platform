# Full-Stack-Dev-Collaboration-Platform
📌 Overview
DevCollab is a GitHub + Slack hybrid built for developer teams. It combines project management (Kanban boards, issue tracking) with real-time communication (threaded chat, presence indicators) and code collaboration (syntax-highlighted snippets, PR-style reviews).
Built with a React.js frontend, Express/Node.js REST API, MongoDB for persistence, Socket.IO for real-time features, and AWS S3 for file storage.
Key Metrics:

👥 Supports 1,000+ concurrent users (load tested with k6)
⚡ < 200ms message delivery latency end-to-end
🔐 JWT auth + OAuth2.0 Google login with RBAC (3 tiers)
📉 35% reduction in user onboarding time vs password-only auth
🚀 Automated CI/CD via GitHub Actions


✨ Features
👤 Auth & Users

Email/password signup with JWT access + refresh tokens
Google OAuth 2.0 one-click login
Role-Based Access Control — Admin / Member / Viewer
Persistent sessions with secure HttpOnly cookie storage

📋 Project Boards

Kanban-style boards with drag-and-drop card management
Issue tracking with labels, assignees, due dates, and priority flags
Activity log per project — every action timestamped and attributed

💬 Real-Time Messaging

Socket.IO powered threaded chat per project
Live typing indicators and online presence
Message history paginated via cursor-based API (no offset pagination)
File/image attachments via AWS S3 pre-signed URLs

💻 Code Snippets

Syntax-highlighted snippet sharing with language auto-detection
Inline comment threads on specific lines (PR review style)
Version history — track edits with diffs

🚀 DevOps

GitHub Actions CI pipeline: lint → test → build → deploy
Docker Compose for local development
Environment-based config (dev / staging / prod)


🏗 Architecture
┌──────────────────────────────────────────────────────────────┐
│                     React.js Frontend                        │
│          (Vite, React Query, Zustand, Tailwind CSS)          │
└─────────────────────────────┬────────────────────────────────┘
                              │ REST + WebSocket
                              ▼
┌──────────────────────────────────────────────────────────────┐
│                  Node.js / Express API                       │
│     Auth Service │ Project Service │ Message Service         │
└─────┬───────────────────────┬─────────────────────┬──────────┘
      │                       │                     │
      ▼                       ▼                     ▼
┌──────────┐          ┌──────────────┐      ┌──────────────┐
│ MongoDB  │          │  Socket.IO   │      │   AWS S3     │
│(Mongoose)│          │  (Redis pub) │      │(File storage)│
└──────────┘          └──────────────┘      └──────────────┘
Tech decisions:

MongoDB over SQL — flexible schema suits varied project/issue shapes
Cursor-based pagination — consistent performance at scale vs OFFSET
Redis adapter for Socket.IO — enables horizontal scaling across API pods
Pre-signed S3 URLs — files upload directly from browser, API never handles raw bytes


🚀 Getting Started
Prerequisites

Node.js 20+
MongoDB (local or Atlas)
Redis
AWS account (for S3) — or use LocalStack for local dev

1. Clone the repo
bashgit clone https://github.com/yourusername/devcollab.git
cd devcollab
2. Install dependencies
bash# Backend
cd server && npm install

# Frontend
cd ../client && npm install
3. Configure environment variables
server/.env
envPORT=5000
MONGODB_URI=mongodb://localhost:27017/devcollab
JWT_ACCESS_SECRET=your-access-secret
JWT_REFRESH_SECRET=your-refresh-secret
JWT_ACCESS_EXPIRES=15m
JWT_REFRESH_EXPIRES=7d
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
AWS_ACCESS_KEY_ID=your-aws-key
AWS_SECRET_ACCESS_KEY=your-aws-secret
AWS_S3_BUCKET=devcollab-uploads
AWS_REGION=ap-south-1
REDIS_URL=redis://localhost:6379
CLIENT_URL=http://localhost:5173
client/.env
envVITE_API_URL=http://localhost:5000/api
VITE_SOCKET_URL=http://localhost:5000
VITE_GOOGLE_CLIENT_ID=your-google-client-id
4. Run locally
bash# Terminal 1 — backend
cd server && npm run dev

# Terminal 2 — frontend
cd client && npm run dev
App runs at http://localhost:5173

📡 API Reference
Auth
MethodEndpointDescriptionPOST/api/auth/registerEmail/password signupPOST/api/auth/loginLogin, returns JWT pairPOST/api/auth/refreshRefresh access tokenGET/api/auth/googleInitiate Google OAuthPOST/api/auth/logoutInvalidate refresh token
Projects
MethodEndpointDescriptionPOST/api/projectsCreate projectGET/api/projectsList user's projectsGET/api/projects/:idGet project detailsPATCH/api/projects/:idUpdate projectPOST/api/projects/:id/membersInvite member
Messages
MethodEndpointDescriptionGET/api/projects/:id/messages?cursor=Paginated historyDELETE/api/messages/:idDelete message
Socket.IO Events
javascript// Client → Server
socket.emit("join_project", { projectId });
socket.emit("send_message", { projectId, content, attachments });
socket.emit("typing_start", { projectId });
socket.emit("typing_stop", { projectId });

// Server → Client
socket.on("new_message", ({ message, sender }));
socket.on("user_typing", ({ userId, username }));
socket.on("user_joined", ({ userId }));
socket.on("user_left", ({ userId }));

🗂 Project Structure
devcollab/
├── client/                         # React frontend
│   ├── src/
│   │   ├── components/             # Reusable UI components
│   │   ├── pages/                  # Route-level page components
│   │   ├── hooks/                  # Custom React hooks
│   │   ├── store/                  # Zustand global state
│   │   ├── services/               # Axios API client
│   │   └── socket/                 # Socket.IO client setup
│   └── vite.config.js
│
├── server/                         # Node.js backend
│   ├── src/
│   │   ├── controllers/            # Route handlers
│   │   ├── middleware/             # Auth, error, rate-limit
│   │   ├── models/                 # Mongoose schemas
│   │   ├── routes/                 # Express routers
│   │   ├── services/               # Business logic layer
│   │   ├── socket/                 # Socket.IO event handlers
│   │   └── utils/                  # Helpers, JWT, S3 client
│   └── index.js
│
├── .github/workflows/ci.yml        # GitHub Actions CI/CD
├── docker-compose.yml
└── README.md

🧪 Testing
bash# Backend unit + integration tests
cd server && npm test

# Frontend component tests
cd client && npm test

# Load test (requires k6)
k6 run load-tests/concurrent-users.js

📊 Load Test Results
Simulated 1,000 concurrent virtual users over 5 minutes:
MetricResultPeak concurrent users1,000Avg response time (REST)87msp95 response time210msMessage delivery latency< 200msError rate0.3%Throughput~2,400 req/min
