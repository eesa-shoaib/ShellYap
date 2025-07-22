# Shell Yap Stage 1 - Simplified Architecture

## High-Level Component Design

```
┌──────────────────────────────────────────────────────────────────┐
│                    Shell Yap Terminal Chat                       │
│                         (Single Binary)                          │
├──────────────────────────────────────────────────────────────────┤
│  ┌───────────────┐                    ┌─────────────────────────┐│
│  │   TUI Client  │◄──── WebSocket ───►│      Server             ││
│  │ (Bubble Tea)  │                    │                         ││
│  │               │                    │  ┌─────────────────────┐││
│  │ ┌───────────┐ │                    │  │                     │││
│  │ │Chat Screen│ │                    │  │    App Manager      │││
│  │ │           │ │                    │  │                     │││
│  │ │LoginScreen│ │                    │  │ ┌─────────────────┐ │││
│  │ │           │ │                    │  │ │  Auth Handler   │ │││
│  │ │Help Screen│ │                    │  │ │                 │ │││
│  │ └───────────┘ │                    │  │ │ Message Handler │ │││
│  └───────────────┘                    │  │ │                 │ │││
│                                       │  │ │  User Handler   │ │││
│                                       │  │ └─────────────────┘ │││
│                                       │  └─────────────────────┘││
│                                       │                         ││
│                                       │  ┌─────────────────────┐││
│                                       │  │  WebSocket Server   │││
│                                       │  │                     │││
│                                       │  │ ┌─────────────────┐ │││
│                                       │  │ │Connection Pool  │ │││
│                                       │  │ │                 │ │││
│                                       │  │ │Message Broadcast│ │││
│                                       │  │ └─────────────────┘ │││
│                                       │  └─────────────────────┘││
│                                       │                         ││
│                                       │  ┌─────────────────────┐││
│                                       │  │   PostgreSQL DB     │││
│                                       │  │                     │││
│                                       │  │ ┌─────────────────┐ │││
│                                       │  │ │    Messages     │ │││
│                                       │  │ │                 │ │││
│                                       │  │ │     Users       │ │││
│                                       │  │ │                 │ │││
│                                       │  │ │    Sessions     │ │││
│                                       │  │ └─────────────────┘ │││
│                                       │  └─────────────────────┘││
│                                       └─────────────────────────┘│
└─────────────────────────────────────────────────────────────────┘
```

## Message Flow Diagram

```
┌─────────────┐                    ┌─────────────────────────────────┐
│             │   1. User Input    │                                 │
│ TUI Client  │ ──────────────────►│         Server                  │
│             │                    │                                 │
│             │                    │  ┌─────────────────────────────┐│
│             │                    │  │      App Manager            ││
│             │                    │  │                             ││
│             │                    │  │ 2. Validate & Process       ││
│             │                    │  │    Message                  ││
│             │                    │  └─────────────┬───────────────┘│
│             │                    │                │                │
│             │                    │                ▼                │
│             │                    │  ┌─────────────────────────────┐│
│             │                    │  │      PostgreSQL             ││
│             │                    │  │                             ││
│             │                    │  │ 3. Store Message            ││
│             │                    │  └─────────────┬───────────────┘│
│             │                    │                │                │
│             │                    │                ▼                │
│             │                    │  ┌─────────────────────────────┐│
│             │                    │  │   WebSocket Server          ││
│             │                    │  │                             ││
│             │ 4. Broadcast       │  │ 4. Send to Connected        ││
│             │ ◄──────────────────┼──┤    Clients                  ││
│             │                    │  └─────────────────────────────┘│
│             │                    │                                 │
└─────────────┘                    └─────────────────────────────────┘

Authentication Flow:
┌─────────────┐                    ┌─────────────────────────────────┐
│ TUI Client  │ 1. Login Request   │         Server                  │
│             │ ──────────────────►│                                 │
│             │                    │  ┌─────────────────────────────┐│
│             │                    │  │     Auth Handler            ││
│             │                    │  │                             ││
│             │                    │  │ 2. Verify Credentials       ││
│             │                    │  └─────────────┬───────────────┘│
│             │                    │                ▼                │
│             │                    │  ┌─────────────────────────────┐│
│             │                    │  │      PostgreSQL             ││
│             │                    │  │   (Check User & Password)   ││
│             │                    │  └─────────────┬───────────────┘│
│             │                    │                ▼                │
│             │                    │  ┌─────────────────────────────┐│
│             │ 3. JWT Token       │  │   Create Session Token      ││
│             │ ◄──────────────────┼──┤   Store in Database         ││
│             │                    │  └─────────────────────────────┘│
│             │                    │                                 │
│             │ 4. WebSocket       │                                 │
│             │    Connection      │                                 │
│             │ ◄─────────────────►│                                 │
└─────────────┘                    └─────────────────────────────────┘
```

## Stage 1 Project Structure

```
ShellYap/
├── cmd/
│   ├── server/
│   │   ├── main.go                 # Server entry point
│   │   └── commands/
│   │       ├── root.go             # Root command
│   │       ├── serve.go            # Start server command
│   │       └── migrate.go          # Database migration command
│   │
│   └── client/
│       ├── main.go                 # Client entry point
│       └── commands/
│           ├── root.go             # Root command
│           ├── chat.go             # Start chat command
│           └── connect.go          # Connect to server command
│
├── internal/
│   ├── app/
│   │   ├── app.go                  # Main app manager
│   │   ├── auth.go                 # Authentication logic
│   │   ├── message.go              # Message handling
│   │   ├── user.go                 # User management
│   │   └── types.go                # Core data types
│   │
│   ├── storage/
│   │   ├── interfaces.go           # Storage contracts
│   │   ├── postgres/
│   │   │   ├── postgres.go         # PostgreSQL implementation
│   │   │   ├── migrations.go       # Migration helpers
│   │   │   ├── queries.go          # SQL queries
│   │   │   └── models.go           # Database models
│   │   │
│   │   └── memory/                 # In-memory storage for testing
│   │       └── memory.go
│   │
│   ├── transport/
│   │   ├── interfaces.go           # Transport contracts
│   │   └── websocket/
│   │       ├── server.go           # WebSocket server
│   │       ├── client.go           # WebSocket client
│   │       ├── hub.go              # Connection hub
│   │       └── message.go          # Message protocol
│   │
│   ├── tui/
│   │   ├── app.go                  # Main TUI application
│   │   ├── models/
│   │   │   ├── login.go            # Login screen model
│   │   │   ├── chat.go             # Chat screen model
│   │   │   ├── help.go             # Help screen model
│   │   │   ├── setting.go          # Setting screen model
│   │   │   └── common.go           # Shared model utilities
│   │   │
│   │   ├── components/
│   │   │   ├── input.go            # Input component
│   │   │   ├── messages.go         # Message list component
│   │   │   └── status.go           # Status bar component
│   │   │
│   │   └── styles/
│   │       ├── styles.go           # Base styles
│   │       └── colors.go           # Color definitions
│   │
│   └── config/
│       ├── config.go               # Configuration struct
│       └── loader.go               # Configuration loader
│
├── migrations/
│   ├── 001_initial_schema.sql      # Initial database schema
│   ├── 002_add_indexes.sql         # Performance indexes
│   └── 003_add_sessions.sql        # Session management
│
├── configs/
│   ├── server.yaml                 # Server configuration
│   └── client.yaml                 # Client configuration
│
├── test/
│   ├── integration/                # Integration tests
│   └── fixtures/                   # Test data
│
├── docker/
│
├── docs/
│   ├── api/                        # Server container
│   ├── architecture/               # Client container
│   └── development/                # Development setup
│
├── go.mod
├── go.sum
├── Makefile                        # Build and development tasks
└── README.md
```

## Key Simplifications in Stage 1

### 1. Single App Manager
- Combines Message, User, and Session management
- Simplified interface with focused methods
- Direct database interaction without caching layer

### 2. PostgreSQL Only
- No Redis dependency
- Uses PostgreSQL's LISTEN/NOTIFY for real-time features
- Sessions stored in database with proper indexing

### 3. WebSocket Transport Only
- Single, well-tested transport protocol
- Simplified connection management
- Focus on stability over extensibility

### 4. Basic TUI
- Three core screens: Login, Chat, Help
- Simple, functional UI without themes
- Keyboard-focused interaction

### 5. Minimal Configuration
- YAML configuration files
- Environment variable overrides
- Sensible defaults for quick setup

## Core Data Models for Stage 1

```go
// User represents a chat user
type User struct {
    ID       string    `json:"id" db:"id"`
    Username string    `json:"username" db:"username"`
    Password string    `json:"-" db:"password_hash"`
    JoinedAt time.Time `json:"joined_at" db:"joined_at"`
}

// Message represents a chat message
type Message struct {
    ID        string    `json:"id" db:"id"`
    UserID    string    `json:"user_id" db:"user_id"`
    Content   string    `json:"content" db:"content"`
    Timestamp time.Time `json:"timestamp" db:"timestamp"`
    Username  string    `json:"username" db:"username"` // Joined field
}

// Session represents a user session
type Session struct {
    Token     string    `json:"token" db:"token"`
    UserID    string    `json:"user_id" db:"user_id"`
    CreatedAt time.Time `json:"created_at" db:"created_at"`
    ExpiresAt time.Time `json:"expires_at" db:"expires_at"`
}
```
