# Shell Yap - Terminal Chat Application

> A modern, real-time terminal-based chat application built with Go, PostgreSQL, and WebSocket technology.
> Passion project to have a chat application that integrates with in one's development environment.

## Features

- **Real-time messaging** with WebSocket communication
- **Terminal UI** powered by Bubble Tea framework
- **Multi-user support** with session management
- **Message persistence** with PostgreSQL
- **Simple authentication** with JWT tokens
- **Cross-platform** support (Linux, macOS, Windows)
- **Lightweight** single binary distribution

## Table of Contents

- [Quick Start](#-quick-start)
- [Installation](#-installation)
  - [Pre-built Binaries](#pre-built-binaries)
  - [Build from Source](#build-from-source)
  - [Using Docker](#using-docker)
- [Configuration](#-configuration)
  - [Server Configuration](#server-configuration)
  - [Client Configuration](#client-configuration)
  - [Environment Variables](#environment-variables)
- [Usage](#-usage)
  - [Starting the Server](#starting-the-server)
  - [Connecting with Client](#connecting-with-client)
  - [Commands Reference](#commands-reference)
- [Architecture](#-architecture)
  - [System Overview](#system-overview)
  - [Database Schema](#database-schema)
  - [WebSocket Protocol](#websocket-protocol)
- [Development](#-development)
  - [Prerequisites](#prerequisites)
  - [Local Development Setup](#local-development-setup)
  - [Running Tests](#running-tests)
  - [Database Migrations](#database-migrations)
- [API Documentation](#-api-documentation)
  - [WebSocket Messages](#websocket-messages)
  - [Authentication Flow](#authentication-flow)
- [Troubleshooting](#-troubleshooting)
  - [Common Issues](#common-issues)
  - [Connection Problems](#connection-problems)
  - [Database Issues](#database-issues)
- [Contributing](#-contributing)
  - [Code Guidelines](#code-guidelines)
  - [Pull Request Process](#pull-request-process)
  - [Issue Templates](#issue-templates)
- [Security](#-security)
  - [Security Policy](#security-policy)
  - [Reporting Vulnerabilities](#reporting-vulnerabilities)
- [Roadmap](#-roadmap)
  - [Current Version (v1.0)](#current-version-v10)
  - [Planned Features](#planned-features)
  - [Future Enhancements](#future-enhancements)
- [FAQ](#-faq)
- [License](#-license)
- [Acknowledgments](#-acknowledgments)

## Quick Start

```bash
# 1. Start PostgreSQL database
docker run -d --name shellyap-db \
  -e POSTGRES_DB=shellyap \
  -e POSTGRES_USER=shellyap \
  -e POSTGRES_PASSWORD=password \
  -p 5432:5432 postgres:15

# 2. Download and run server
./shellyap-server serve --port 8080

# 3. Connect with client (in another terminal)
./shellyap-client connect --server localhost:8080
```

## Installation

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DB_URL` | PostgreSQL connection string | Required |
| `SERVER_PORT` | Server port number | `8080` |
| `JWT_SECRET` | JWT signing secret | Required |
| `LOG_LEVEL` | Logging level (debug, info, warn, error) | `info` |

## Usage

### Starting the Server

### Connecting with Client

### Commands Reference

#### Server Commands
- `serve` - Start the chat server
- `migrate` - Run database migrations
- `version` - Show version information

#### Client Commands
- `connect` - Connect to chat server
- `version` - Show version information

#### In-Chat Commands
- `/help` - Show available commands
- `/quit` - Exit the application
- `/users` - List online users
- `/clear` - Clear chat history

## Architecture

### System Overview

Shell Yap follows a client-server architecture with real-time WebSocket communication:

- **Client**: Terminal UI built with Bubble Tea
- **Server**: Go HTTP/WebSocket server
- **Database**: PostgreSQL for persistence
- **Protocol**: JSON over WebSocket

### Database Schema

```sql
-- Users table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    username VARCHAR(50) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    joined_at TIMESTAMP DEFAULT NOW()
);

-- Messages table
CREATE TABLE messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    content TEXT NOT NULL,
    timestamp TIMESTAMP DEFAULT NOW()
);

-- Sessions table
CREATE TABLE sessions (
    token VARCHAR(255) PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    created_at TIMESTAMP DEFAULT NOW(),
    expires_at TIMESTAMP NOT NULL
);
```

### WebSocket Protocol

Messages are exchanged as JSON objects:

```json
{
  "type": "message",
  "data": {
    "content": "Hello, world!",
    "timestamp": "2024-01-01T12:00:00Z"
  }
}
```

## Development

### Prerequisites

- Go 1.21 or higher
- PostgreSQL 13 or higher
- Make (for build automation)

### Local Development Setup

```bash
# Clone repository
git clone https://github.com/yourusername/shellyap.git
cd shellyap

# Install dependencies
go mod download

# Start PostgreSQL (using Docker)

# Run database migrations

# Start development server

# In another terminal, start client
```

### Running Tests

### Database Migrations

## 📚 API Documentation

### WebSocket Messages

#### Message Types

| Type | Direction | Description |
|------|-----------|-------------|
| `auth` | Client → Server | Authentication request |
| `message` | Bidirectional | Chat message |
| `user_joined` | Server → Client | User joined notification |
| `user_left` | Server → Client | User left notification |
| `error` | Server → Client | Error message |

#### Authentication Flow

1. Client connects to WebSocket endpoint
2. Server sends auth challenge
3. Client responds with credentials
4. Server validates and sends JWT token
5. Client uses token for subsequent requests

### WebSocket Examples

```json
// Client authentication
{
  "type": "auth",
  "data": {
    "username": "alice",
    "password": "secret123"
  }
}

// Server response
{
  "type": "auth_success",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs...",
    "user_id": "123e4567-e89b-12d3-a456-426614174000"
  }
}

// Send message
{
  "type": "message",
  "data": {
    "content": "Hello everyone!"
  }
}
```

##  Troubleshooting

### Common Issues

### Connection Problems

### Database Issues

### Code Guidelines

- Follow [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- Add tests for new features
- Update documentation as needed
- Use conventional commit messages

### Pull Request Process

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests and documentation
5. Submit a pull request

## Security

### Security Policy

### Reporting Vulnerabilities

**Do not open public issues for security vulnerabilities.**

## Roadmap

### Current Version (v1.0)
- Real-time messaging
- User authentication
- Terminal UI
- PostgreSQL persistence

### Planned Features
- Message editing and deletion
- File sharing
- User profiles and avatars
- Chat rooms and channels

### Future Enhancements
- End-to-end encryption
- Redis caching layer
- Plugin system
- Message search
- Emoji and sticker support

## FAQ

**Q: Can I run Shell Yap on Windows?**
A: Yes, Shell Yap supports Windows, Linux, and macOS.

**Q: Does Shell Yap support multiple chat rooms?**
A: Currently, Shell Yap supports a single global chat room. Multiple rooms are planned for v2.0.

**Q: How do I backup my chat history?**
A: Use PostgreSQL dump tools to backup your database

## License

## Acknowledgments

- [Bubble Tea](https://github.com/charmbracelet/bubbletea) - Terminal UI framework
- [Gorilla WebSocket](https://github.com/gorilla/websocket) - WebSocket implementation
- [Cobra](https://github.com/spf13/cobra) - CLI framework
- [PostgreSQL](https://postgresql.org) - Database system

---

**Made with ❤️ by the Shell Yap team**
