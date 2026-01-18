# AI Interviewer Platform


**An intelligent, modern AI-powered interview management and execution platform**

## Overview

**AI Interviewer** is a comprehensive platform designed to streamline the interview process using cutting-edge AI technology. It enables companies to conduct intelligent, scalable interviews with candidates while providing detailed performance analytics, resume analysis, and AI-powered feedback.

### Key Highlights
- **AI-Powered Interviews**: Intelligent question generation and response evaluation using Google's Generative AI
- **Comprehensive Analytics**: Performance scoring, strength/weakness analysis, and role recommendations
- **Multi-Role Support**: Separate interfaces for admins and candidates with role-based access control
- **Resume Analysis**: Automatic resume parsing and skill extraction using LangChain
- **Real-time Collaboration**: WebSocket-based live interview sessions with Redis caching
- **Secure Authentication**: NextAuth integration with JWT token management
- **Cloud Storage**: Backblaze B2 integration for resume and media storage
- **Voice Integration**: LiveKit SDK for real-time voice communication

---

## Features

### For Admins
- **Candidate Management**
  - Add, view, and manage candidate profiles
  - Upload and parse resumes with automatic skill extraction
  - Track candidate status (New, Interview Scheduled, Interview Completed)
  - Bulk operations for efficient workflow

- **Interview Management**
  - Create and schedule interviews for specific job positions
  - Generate AI-powered interview questions
  - Real-time interview monitoring and control
  - Reschedule or cancel interviews with detailed reasons
  - Access interview recordings and transcripts

- **Job Management**
  - Create job openings with detailed descriptions
  - Define interview criteria and question templates
  - Track job status (Open/Closed)
  - Generate position-specific interview questions

- **Analytics & Reporting**
  - Performance scoring for each candidate
  - Strength and weakness identification
  - Role recommendations based on interview performance
  - Comprehensive interview history and statistics

### For Candidates
- **Interview Portal**
  - View scheduled interviews with details
  - Join interviews seamlessly
  - Real-time interview experience with AI interaction
  - View interview status and outcomes

- **Profile Management**
  - Upload and manage resume
  - Track interview history (Upcoming, Completed, Cancelled)
  - View detailed interview feedback
  - Performance metrics and recommendations

---

## Tech Stack

### Frontend
| Technology | Purpose |
|-----------|---------|
| **Next.js 16** | React framework with SSR, API routes, and file-based routing |
| **React 19** | UI library for building interactive interfaces |
| **Tailwind CSS 4** | Utility-first CSS framework with PostCSS support |
| **Framer Motion** | Animation library for smooth, professional transitions |
| **Socket.io Client** | Real-time bidirectional communication |
| **LiveKit Client** | WebRTC for real-time video/audio sessions |
| **React Quill** | Rich text editor for job descriptions and feedback |
| **NextAuth** | Authentication and session management |
| **React Toastify** | Toast notifications and alerts |
| **Lucide React** | Modern icon library |
| **date-fns** | Date manipulation and formatting |
| **Backblaze B2** | Cloud storage for resumes and files |

### Backend
| Technology | Purpose |
|-----------|---------|
| **Motia Framework** | Advanced Node.js framework with built-in features |
| **Express.js** | HTTP server and routing (via Motia) |
| **Prisma ORM** | Type-safe database client and migrations |
| **PostgreSQL** | Relational database with Prisma adapter |
| **Google Generative AI** | LLM for question generation and analysis |
| **LangChain** | AI orchestration and prompt management |
| **LiveKit SDK** | Server-side room and token generation |
| **Socket.io** | Real-time event handling |
| **Redis** | Caching and session management |
| **BullMQ** | Job queue for async tasks |
| **Nodemailer** | Email notifications |
| **PDF Parse** | Resume text extraction |
| **bcrypt** | Password hashing and security |
| **JWT** | Token-based authentication |

### Infrastructure & DevOps
- **Node.js** with ES Modules
- **PostgreSQL** with Prisma migrations
- **Redis** for caching and pub/sub
- **BullMQ** for job scheduling
- **Ngrok** for development tunneling

---

## System Architecture

### Layered Architecture Pattern

```
PRESENTATION LAYER
├─ Next.js Client (Port 3001)
│  ├─ Admin Dashboard (React Components)
│  ├─ Candidate Portal (React Components)
│  └─ Authentication UI (NextAuth)

BUSINESS LOGIC LAYER
├─ API Server (Motia, Port 8000)
│  ├─ AdminService
│  ├─ AuthService
│  ├─ CandidateService
│  └─ CronService
│
└─ Real-time Server (Socket.io, Port 3000)
   └─ Event Handlers & Broadcasting

PERSISTENCE LAYER
├─ PostgreSQL (Database)
├─ Redis (Cache & Pub/Sub)
└─ Backblaze B2 (File Storage)

EXTERNAL SERVICES
├─ Google Generative AI (LLM)
├─ LiveKit (Voice/Video)
└─ Nodemailer (Email)
```

### Component Interaction Map

```
CLIENT (Next.js + React)
    │
    ├────────────────────────┬──────────────────────┐
    │                        │                      │
    ▼                        ▼                      ▼
[HTTP API]          [WebSocket/Socket.io]    [NextAuth]
(Port 8000)          (Port 3000)             (Built-in)
    │                        │                      │
    │                        ▼                      │
    │               [Real-time Events]             │
    │                   (Interview Data)           │
    │                                              │
    └────────────────┬────────────────────────────┘
                     │
    ┌────────────────┴────────────────┐
    ▼                                 ▼
[PostgreSQL]                    [Redis Cache]
(Persistent Data)              (Session + State)
    │                                 │
    ├─────────────────────────────────┤
    │                                 │
    ├─ Users & Roles                 ├─ Socket Sessions
    ├─ Candidates                    ├─ Interview State
    ├─ Interviews                    ├─ User Cache
    ├─ Questions & Feedback          └─ Job Queue State
    ├─ Resume Profiles               
    └─ Job Postings         [Backblaze B2]
                           (File Storage)
                                 │
                            ┌────┴──────┐
                            │            │
                       [Resumes]    [Media]
```

---

## Data Flow Diagrams

### Authentication Flow

```
┌─────────────────────────────────────────────────────────────────┐
│ USER REGISTRATION/LOGIN FLOW                                    │
└─────────────────────────────────────────────────────────────────┘

1. User Submission
   Client (SignupForm.jsx)
       │
       └─> POST /auth/signup { email, password, role, ... }
           │
           ▼
   Server (signup.js endpoint)
       │
       ├─> Validate input (Zod)
       ├─> Hash password (bcrypt)
       ├─> Create User + Admin/Candidate record (Prisma)
       └─> Return JWT tokens
           │
           ▼
   Client (NextAuth)
       │
       ├─> Store tokens (HTTP-only cookies)
       ├─> Update UserContext
       └─> Redirect to dashboard
           │
           ▼
   Protected Routes
       │
       ├─> Check auth.middleware.js
       ├─> Verify JWT
       └─> Grant access to protected resource


┌─────────────────────────────────────────────────────────────────┐
│ API REQUEST WITH AUTHENTICATION                                 │
└─────────────────────────────────────────────────────────────────┘

Client Request:
    │
    ├─ GET /admin/candidates
    ├─ Header: Authorization: Bearer <JWT_TOKEN>
    └─ Header: Content-Type: application/json
        │
        ▼
Server Processing:
    │
    ├─ auth.middleware.js
    │   ├─ Extract token from header
    │   ├─ Verify signature (jwtToken.util.js)
    │   ├─ Check expiration
    │   └─ Attach user to request
    │
    ├─ Route Handler (candidates.js)
    │   ├─ Access request.user
    │   ├─ Fetch data (AdminService)
    │   └─ Return response
    │
    └─ Response: 200 OK
        │
        ▼
Client:
    │
    ├─ Receive response
    ├─ Update component state
    └─ Re-render UI
```

### Interview Scheduling & Execution Flow

```
┌─────────────────────────────────────────────────────────────────┐
│ INTERVIEW LIFECYCLE                                             │
└─────────────────────────────────────────────────────────────────┘

PHASE 1: CREATION & SCHEDULING
─────────────────────────────

Admin Portal:
    │
    ├─ Select Candidate
    ├─ Select Job Position
    ├─ Set Date & Duration
    └─ Click "Schedule Interview"
        │
        ▼
POST /admin/interviews { candidateId, jobId, scheduledAt, durationMin }
        │
        ▼
AdminService.scheduleInterview()
    │
    ├─ Validate inputs
    ├─ Create Interview record (status: PENDING)
    ├─ Create initial InterviewProfile
    ├─ Queue email notification
    └─ Return interview details
        │
        ▼
Socket.io Broadcast
    │
    ├─ Notify candidate in real-time
    └─ Candidate app updates interview list
        │
        ▼
Cron Job (24h before)
    │
    ├─ CronService.sendInterviewReminders()
    └─ Send email reminder to candidate


PHASE 2: INTERVIEW EXECUTION
──────────────────────────

Candidate Portal:
    │
    ├─ Views scheduled interview
    ├─ Clicks "Join Interview"
    └─ Calls POST /candidate/interviews/:id/join
        │
        ▼
CandidateService.startInterview()
    │
    ├─ Verify interview status (PENDING)
    ├─ Update status to ONGOING
    ├─ Create LiveKit room & token
    ├─ Initialize Socket.io connection
    ├─ Load job description & context
    └─ Return interview session details
        │
        ▼
Candidate Client:
    │
    ├─ Connect to LiveKit (voice/video)
    ├─ Connect to Socket.io (real-time events)
    └─ Display interview interface
        │
        ▼
Server (CandidateService.conductInterview())
    │
    ├─ Initialize AI context
    ├─ Generate initial questions (Gemini)
    └─ Socket.emit('question_sent', questionData)
        │
        ▼
Real-time Question Flow:
    │
    ├─ [1] Server generates question (Gemini)
    ├─ [2] Socket.io sends to candidate
    ├─ [3] Candidate hears/reads question
    ├─ [4] Candidate speaks/types answer
    ├─ [5] Socket.io sends answer back
    ├─ [6] Server analyzes (Gemini)
    │   ├─ Evaluate correctness
    │   ├─ Score response
    │   └─ Generate feedback
    ├─ [7] Generate follow-up or next question
    └─ Repeat until duration expires or completion
        │
        ▼
Interview Completion:
    │
    ├─ Check duration limit reached
    ├─ Call end_interview_session()
    ├─ Calculate performance score
    ├─ Generate recommendations
    ├─ Save InterviewProfile with results
    ├─ Update Interview status to COMPLETED
    ├─ Queue email with results
    └─ Socket.io notify both parties


PHASE 3: RESULTS & FEEDBACK
──────────────────────────

Admin Dashboard:
    │
    ├─ Views interview results
    │   ├─ Performance score
    │   ├─ Strengths identified
    │   ├─ Weaknesses identified
    │   └─ Role recommendations
    │
    └─ Can download/share report

Candidate Portal:
    │
    ├─ Views results
    ├─ Reads AI feedback
    ├─ Sees recommendations
    └─ Interview moves to "Completed" section
```

### AI Interview Question Generation & Response Flow

```
┌─────────────────────────────────────────────────────────────────┐
│ GEMINI AI INTEGRATION FOR INTERVIEWS                            │
└─────────────────────────────────────────────────────────────────┘

Context Preparation:
    │
    ├─ Fetch Job description
    ├─ Load Candidate resume profile
    ├─ Get previous questions & answers
    ├─ Analyze candidate skill level
    └─ Build conversation history


Question Generation:
    │
    ├─ CandidateService.conductInterview()
    │   │
    │   └─ gemini.util.js (generateContent)
    │       │
    │       ├─ System prompt:
    │       │  "You are an expert interviewer for [JOB POSITION]"
    │       │
    │       ├─ User prompt:
    │       │  "Generate next interview question based on:"
    │       │  - Job requirements
    │       │  - Candidate experience
    │       │  - Previous answers
    │       │  - Follow-up needed areas
    │       │
    │       └─ Response format (JSON):
    │          {
    │            "question": "What is your experience with X?",
    │            "difficulty": "MEDIUM",
    │            "category": "TECHNICAL",
    │            "expectedKeyPoints": [...]
    │          }
    │
    ├─ Socket.io emit to candidate
    └─ Candidate sees/hears question


Response Analysis:
    │
    ├─ Candidate provides answer
    ├─ Socket.io sends back to server
    │
    └─ gemini.util.js (analyzeResponse)
        │
        ├─ Function calling:
        │  {
        │    "name": "analyze_response",
        │    "parameters": {
        │      "question": "...",
        │      "candidateAnswer": "...",
        │      "expectedKeyPoints": [...]
        │    }
        │  }
        │
        ├─ Gemini response includes:
        │  {
        │    "score": 85,                    # 0-100
        │    "feedback": "Good understanding...",
        │    "weakPoints": ["Could mention..."],
        │    "followUpNeeded": true,
        │    "nextTopic": "Database design"
        │  }
        │
        └─ Save to InterviewQuestion record


Adaptive Flow:
    │
    ├─ Based on score:
    │   │
    │   ├─ If score >= 80: Move to next topic
    │   ├─ If 60-80: Generate follow-up
    │   ├─ If < 60: Simpler question from same topic
    │   │
    │   └─ Adjust difficulty dynamically
    │
    └─ Repeat until interview end


Final Scoring:
    │
    ├─ Aggregate all question scores
    ├─ Calculate performance metrics:
    │   ├─ Overall score (weighted average)
    │   ├─ Topic-wise breakdown
    │   ├─ Consistency analysis
    │   └─ Growth trajectory
    │
    └─ Generate recommendations:
        ├─ Suitable roles
        ├─ Strengths to highlight
        ├─ Areas for improvement
        └─ Suggested learning paths
```

---

## Architecture

### High-Level Architecture

```
┌─────────────────────┐
│   Client (Next.js)  │
│  - Admin Dashboard  │
│  - Candidate Portal │
└──────────┬──────────┘
           │
           ├─────────────────────────┐
           │                         │
    ┌──────▼──────┐        ┌────────▼────────┐
    │  Main API   │        │ Socket Server   │
    │  (Motia)    │        │  (Real-time)    │
    │  Port 8000  │        │   Port 3000     │
    └──────┬──────┘        └────────┬────────┘
           │                         │
           └──────────────┬──────────┘
                          │
          ┌───────────────┼───────────────┐
          │               │               │
     ┌────▼─────┐   ┌────▼─────┐   ┌────▼──────┐
     │PostgreSQL│   │  Redis   │   │ Backblaze│
     │Database  │   │ (Cache)  │   │    B2    │
     └──────────┘   └──────────┘   └──────────┘

External Services:
┌──────────────────┐  ┌──────────────┐  ┌───────────────┐
│ Google Generative│  │   LiveKit    │  │   Nodemailer  │
│      AI (LLM)    │  │   (Voice)    │  │   (Email)     │
└──────────────────┘  └──────────────┘  └───────────────┘
```
---

## Getting Started

### Prerequisites
- **Node.js** 18+ with npm or yarn
- **PostgreSQL** 12+ database
- **Redis** for caching and session management
- **Google Cloud Project** with Generative AI API enabled
- **Backblaze B2** account for cloud storage
- **LiveKit** account for real-time communication
- **SMTP Server** for email (or Gmail for development)

### Installation

#### 1. Clone the Repository
```bash
git clone https://github.com/yourusername/ai-interviewer-nextjs-motia.git
cd ai-interviewer-nextjs-motia
```

#### 2. Set Up Environment Variables

**Server** (`server/.env`):
```bash
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/ai_interviewer

# Redis
REDIS_URL=redis://localhost:6379

# Google AI
GOOGLE_AI_API_KEY=your_google_api_key

# JWT & Security
JWT_SECRET=your_jwt_secret_key
REFRESH_TOKEN_SECRET=your_refresh_secret

# Backblaze B2
BACKBLAZE_APPLICATION_KEY=your_app_key
BACKBLAZE_APPLICATION_KEY_ID=your_app_key_id
BACKBLAZE_BUCKET_NAME=your_bucket_name

# LiveKit
LIVEKIT_API_KEY=your_livekit_api_key
LIVEKIT_API_SECRET=your_livekit_secret
LIVEKIT_URL=your_livekit_url

# Email
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your_email@gmail.com
SMTP_PASSWORD=your_app_password

# Server
NODE_ENV=development
PORT=8000

# Frontend URL
FRONTEND_URL=http://localhost:3001
```

**Client** (`client/.env.local`):
```bash
NEXT_PUBLIC_API_URL=http://localhost:8000
NEXT_PUBLIC_SOCKET_URL=http://localhost:3000

NEXTAUTH_URL=http://localhost:3001
NEXTAUTH_SECRET=your_nextauth_secret

NEXT_PUBLIC_LIVEKIT_URL=your_livekit_url
```

**Socket Server** (`socket-server/.env`):
```bash
REDIS_URL=redis://localhost:6379
JWT_SECRET=your_jwt_secret_key
PORT=3000
```

#### 3. Install Dependencies

```bash
# Install server dependencies
cd server
npm install

# Install client dependencies
cd ../client
npm install

# Install socket server dependencies
cd ../socket-server
npm install
```

#### 4. Set Up Database

```bash
cd server
# Create and apply Prisma migrations
npx prisma migrate deploy

# Seed database (if seed file exists)
npx prisma db seed
```

#### 5. Generate Prisma Client

```bash
cd server
npx prisma generate
```

---

## Running the Application

### Development Mode

**Terminal 1 - Backend Server (Port 8000)**
```bash
cd server
npm run dev
```

**Terminal 2 - Socket Server (Port 3000)**
```bash
cd socket-server
npm start
```

**Terminal 3 - Frontend (Port 3001)**
```bash
cd client
npm run dev
```

Visit `http://localhost:3001` in your browser.


## Key Features Implementation

### 1. AI Interview Generation
The platform uses Google's Generative AI to:
- Generate relevant interview questions based on job description
- Analyze candidate responses in real-time
- Provide intelligent feedback and scoring
- Recommend suitable roles based on performance

### 2. Resume Analysis
- Automatic text extraction from PDF resumes
- Skill extraction and categorization
- Education and experience parsing
- Job area matching and recommendations

### 3. Real-time Interview Sessions
- WebSocket-based live communication
- LiveKit integration for voice/video
- Real-time question delivery and response capture
- Instant AI-powered feedback

### 4. Performance Analytics
- Candidate scoring algorithm
- Strength and weakness identification
- Role recommendations based on performance
- Comprehensive interview reports
