# `ShellYap` Terminal Chat Application

## Architecture Philosophy

The hybrid monolith approach strikes an optimal balance between development simplicity and architectural flexibility.
This design enables rapid iteration during initial development while providing clear extension points for future features
like peer-to-peer communication, alternative transport protocols, and third-party integrations.

The architecture emphasizes **interface-driven design** where core components communicate through well-defined interfaces,
allowing for easy substitution of implementations without affecting the broader system.

## System Architecture

### High-Level Component Design

```
┌─────────────────────────────────────────────────────────────────┐
│                    Terminal Chat Application                    │
│                         (Single Binary)                         │
├─────────────────────────────────────────────────────────────────┤
│  ┌───────────────┐    ┌─────────────────┐    ┌─────────────────┐│
│  │   TUI Layer   │    │  Command Layer  │    │  Plugin System  ││
│  │ (Bubble Tea)  │    │    (Cobra)      │    │  (Interfaces)   ││
│  └───────┬───────┘    └─────────┬───────┘    └─────────┬───────┘│
│          │                      │                      │        │
│  ┌───────┴──────────────────────┴──────────────────────┴──────┐ │
│  │                  Application Core                          │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │ │
│  │  │   Message   │  │    User     │  │     Session         │ │ │
│  │  │   Manager   │  │   Manager   │  │     Manager         │ │ │
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘ │ │
│  └─────────────────────────┬──────────────────────────────────┘ │
│                            │                                    │
│  ┌─────────────────────────┴──────────────────────────────────┐ │
│  │                 Transport Layer                            │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │ │
│  │  │  WebSocket  │  │ SSH Plugin  │  │    Future P2P       │ │ │
│  │  │  (Primary)  │  │ (Optional)  │  │    (Extensible)     │ │ │
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘ │ │
│  └─────────────────────────┬──────────────────────────────────┘ │
│                            │                                    │
│  ┌─────────────────────────┴──────────────────────────────────┐ │
│  │                 Storage Layer                              │ │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │ │
│  │  │ PostgreSQL  │  │    Redis    │  │   Local Storage     │ │ │
│  │  │ (Messages,  │  │ (Sessions,  │  │  (Avatars, Files,   │ │ │
│  │  │  Users,     │  │  Queues,    │  │     Configs)        │ │ │
│  │  │  Search)    │  │  Cache)     │  │                     │ │ │
│  │  └─────────────┘  └─────────────┘  └─────────────────────┘ │ │
│  └────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

### Core Design Principles

**Modular Boundaries**: While contained in a single binary, the application maintains clear module boundaries through Go
packages and interfaces, enabling easy testing and future extraction if needed.

**Plugin-Ready Architecture**: All external communication protocols implement a common `Transport` interface, making it
trivial to add new connection methods without changing core business logic.

**Configuration-Driven**: The application behavior adapts through YAML configuration and environment variables, supporting
different deployment scenarios without code changes.

**Graceful Degradation**: Components like Redis caching are optional, allowing the application to function with reduced
features when external services are unavailable.

**_Application Manager_**

  1. Message Manager

  - `Responsibilities`

    - Sending and receiving messages.
    - Formatting, validating, and routing messages.
    - Handling message history, deletion, and edits.
    - Filtering (e.g., for banned words or user mutes).

  - `Interacts with`

    - WebSocket transport for real-time sending/receiving.
    - PostgreSQL for storing messages.
    - Redis for queueing and broadcasting to active sessions.

  2. User Manager

  - `Responsibilities`

    - Handling user creation, updates, and deletions.
    - Managing display names, avatars, and bios.
    - Validating credentials and user permissions.

  - `Interacts with`

    - PostgreSQL for persistent user info.
    - Local file system for avatar images.
    - Redis (optionally) to cache user data.

  3. Session Manager

  - `Responsibilities`

    - Authenticating users (login/logout, token validation).
    - Maintaining active sessions (who is online, where).
    - Preventing duplicate logins or session hijacking.
    - Timeout or expiration logic.

  - `Interacts with`

    - Redis for fast, in-memory session storage.
    - PostgreSQL for logging session history.


**_Transport Layer_**

  1. `WebSocket Server`

  - Maintains persistent connections with clients.
  - Authenticates the connection via session tokens (checked with Redis).
  - Maps active users to their connections.
  - Sends/receives message packets, user status changes, typing indicators, etc.

  2. `Connection Manager`

  - Tracks all open WebSocket connections.
  - Handles reconnections, timeouts, and dropped clients.
  - Manages broadcasting (e.g., delivering one user’s message to all relevant recipients).
  - Queues delivery to offline users (through Redis or database fallback).
  - SSH for future implementation

  3. `Packet Formatter`

  - Encodes/decodes the message format over the wire (usually JSON or a custom binary protocol).
  - Adds metadata like timestamps, sender ID, message type (chat, system, file, etc.).


**_Storage Layer_**

  1. `PostgreSQL`

  - `Messages:` Stored with timestamps, sender IDs, and content for history/search.
  - `Users:` Contains username, hashed password, metadata, join dates, etc.
  - `Search:` Full-text indexes or trigram similarity for fast querying of messages or usernames.

  2. `Redis` Used as an in-memory store for fast access to frequently changing or ephemeral data.

  - `Sessions:` Store current active session tokens with TTLs.
  - `Queues:` Pub/Sub or task queues (e.g., broadcasting messages to online users).
  - `Cache:` Store user data, recent messages, search results to avoid hitting Postgres too often.

  3. `Local File Storage` Used for binary data and configuration that doesn’t belong in a database.

  - `Avatars:` Images linked to user profiles.
  - `Files:` Attachments users send in chats.
  - `Configs:` Local client-side configuration (like themes or preferences), especially for TUI.


## Project Structure

### Directory Organization

```
terminal-chat/
├── cmd/
│   ├── server/                 # Server entry point and CLI
│   │   ├── main.go
│   │   └── commands/           # Cobra command definitions
│   └── client/                 # Client entry point and CLI
│       ├── main.go
│       └── commands/
├── internal/                   # Private application code
│   ├── app/                   # Application core
│   │   ├── chat/              # Chat business logic
│   │   ├── user/              # User management
│   │   ├── message/           # Message processing
│   │   └── session/           # Session handling
│   ├── transport/             # Communication protocols
│   │   ├── websocket/         # WebSocket implementation
│   │   ├── ssh/               # SSH plugin (future)
│   │   └── interfaces.go      # Transport contracts
│   ├── storage/               # Data persistence
│   │   ├── postgres/          # PostgreSQL implementation
│   │   ├── redis/             # Redis implementation
│   │   ├── filesystem/        # Local file storage
│   │   └── interfaces.go      # Storage contracts
│   ├── tui/                   # Terminal UI components
│   │   ├── models/            # Bubble Tea models
│   │   ├── components/        # Reusable UI components
│   │   └── styles/            # UI styling and themes
│   ├── config/                # Configuration management
│   └── plugin/                # Plugin system framework
├── pkg/                       # Public packages (if needed)
├── api/                       # API definitions (protobuf/openapi)
├── configs/                   # Configuration files
├── migrations/                # Database migrations
├── test/                      # Test utilities and fixtures
├── docker/                    # Docker configurations
└── docs/                      # Documentation
```

### Core Interfaces

* Transport Interface
  - Enables pluggable communication protocols

* Storage Interface
  - Enables pluggable storage backends

* Plugin Interface
  - Enables extensible functionality

## Feature Implementation Strategy

### Real-Time Messaging Core

The messaging system operates through a central message hub that manages all communication flows. Messages are typed
and structured, supporting various content types while maintaining extensibility for future message formats.

**Message Flow Architecture**:
1. TUI captures user input and creates message objects
2. Message manager validates and processes messages
3. Transport layer handles delivery to recipients
4. Storage layer persists messages for history and search
5. Delivery confirmations flow back through the same path

### User Management System

User management encompasses authentication, profile management, and relationship handling (contacts, blocking).
The system maintains user sessions across application restarts through persistent storage.

**Authentication Flow**:
- JWT-based session tokens with configurable expiration
- bcrypt password hashing with adjustable cost factors
- Session persistence through Redis or database fallback
- Automatic session refresh and cleanup processes

### Command System Integration

The Cobra framework provides structured command handling with automatic help generation and command completion.
Commands are organized hierarchically and support both interactive and batch execution modes.

**Command Categories**:
- **Connection Commands**: Login, logout, connect, disconnect
- **Message Commands**: Send, reply, edit, delete, search
- **User Commands**: Profile, contacts, block, unblock
- **System Commands**: Status, settings, help, quit

### TUI Implementation Strategy

Bubble Tea models represent different application states and screens, with smooth transitions between modes.
The interface supports both mouse and keyboard interaction where the terminal allows.

**Screen Architecture**:
- **Login Screen**: Authentication and initial connection
- **Chat Screen**: Primary messaging interface with sidebar
- **Contacts Screen**: User management and contact lists
- **Settings Screen**: Configuration and preferences
- **Help Screen**: Command reference and documentation

### Storage and Caching Strategy

PostgreSQL serves as the source of truth for all persistent data, with Redis providing performance enhancements
through caching and real-time features like message queuing for offline users.

**Data Flow**:
- **Write Operations**: Direct to PostgreSQL with Redis cache invalidation
- **Read Operations**: Redis cache first, PostgreSQL fallback
- **Search Operations**: PostgreSQL full-text search with result caching
- **File Storage**: Local filesystem with configurable storage paths

## Configuration Architecture

### Layered Configuration System

Configuration loading follows a hierarchical approach:
1. **Default Values**: Hardcoded sensible defaults
2. **Configuration Files**: YAML files for structured configuration
3. **Environment Variables**: Override any configuration value
4. **Command Line Flags**: Override specific values for single execution

## Security Implementation

### Authentication and Authorization

JWT tokens carry user identity and permissions, with configurable expiration and automatic refresh capabilities.
The system supports multiple authentication methods through pluggable authentication providers.

### Data Protection

All sensitive data is encrypted at rest using industry-standard encryption algorithms. Connection security is enforced
through TLS for WebSocket connections and SSH for alternative transport methods.

## Testing Strategy (Once the work is done)

- Unit Testing Approach

- Integration Testing Framework

- End-to-End Testing Suite

## Deployment and Operations

- Single Binary Distribution

- Container Strategy

# Extensibility Framework

### UI Customization Points

The TUI framework supports theming, custom key bindings, and layout modifications through configuration files and plugin mechanisms.
Using Toml file

### Plugin Support
Not decided


