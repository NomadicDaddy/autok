# TodoList - Small Full-Stack Todo Application

**Version:** 1.0.0
**Template:** Spernakit v1.7.0
**Stack:** SPERN (SQLite, Prisma, Express, React, Node.js on Bun)

## Overview

Build a small, polished todo list application with a TypeScript/React 19 frontend and a TypeScript/Express backend. The app supports creating, viewing, updating, completing, and deleting todos, plus basic organization (filters + optional lists/tags). The focus is correctness, clean UX, accessible UI, and a simple maintainable API.

## Technology Stack

### Spernakit Architecture

- **Core Stack**: SPERN (SQLite, Prisma, Express, React, Node.js)
- **Runtime**: Bun for both frontend and backend services
- **Package Manager**: Bun for dependency management and scripts
- **Monorepo Structure**: Workspace architecture with frontend/backend separation
- **Configuration**: JSON-driven config via `config/todolist.json` (no implicit .env loading)

### Frontend

- **Framework**: React 19 with Vite 5
- **Styling**: Tailwind CSS with DaisyUI component library
- **State Management**: React hooks; optional lightweight store (Zustand) if needed
- **Routing**: Optional (single-page is fine); React Router if adding multiple views
- **Data Fetching**: TanStack Query (React Query) for server state
- **Testing**: Vitest + React Testing Library
- **Port**: {frontend_port} (default: 3330)

### Backend

- **Runtime**: Node.js with Express 5, running on Bun
- **Database**: SQLite with Prisma ORM (PostgreSQL-compatible schema)
- **Validation**: Zod for request validation
- **Testing**: Vitest or Jest + Supertest
- **Observability**: Request logging + structured error responses
- **Port**: {backend_port} (default: 3331)

### Configuration

All configuration stored in `config/todolist.json`:

- Server ports (frontend: 3330, backend: 3331)
- Database path: `file:../../data/todolist.db`
- Feature flags and environment settings
- CORS and rate limiting settings

## Prerequisites

### Environment Setup

- Application configuration stored in `config/todolist.json`
- Frontend dependencies installed via `bun install` in `/frontend`
- Backend dependencies installed via `bun install` in `/backend`
- Database bootstrapped via `bun run db:setup`
- Development assumes Bun environment with no implicit .env loading

### Development Commands

```bash
bun run init          # Initialize workspaces and install deps
bun run dev           # Start both frontend and backend
bun run dev:backend   # Start Express backend only
bun run dev:frontend  # Start React frontend only
bun run build         # Build both frontend and backend
bun run format        # Format code with Prettier
bun run lint          # Lint code with ESLint
bun run db:setup      # Generate Prisma client, run migrations, seed data
```

## Core Features

### Todo Management

- Create a todo with: title (required), optional description, optional due date
- Edit todo title/description/due date
- Mark complete/incomplete
- Delete todo (with confirmation)
- Persist todos via backend (no localStorage-only solution)
- Optimistic UI for fast interactions with rollback on failure

### Organization

- Filters: All, Active, Completed
- Sort: created_at (default), due_date, title
- Optional: lists/projects (e.g., "Personal", "Work")
- Optional: tags (multi-select) with simple string tags
- Search by title/description

### UI/UX

- Clean layout: header + main list + footer stats
- Keyboard-friendly: Enter to add, Escape to cancel edit
- Inline editing (double click or edit button)
- Clear empty states (no todos yet, no matches for filter/search)
- Loading and error states for all API calls
- Accessible controls with labels, focus management, and ARIA where needed
- Responsive: works well on mobile widths

### Quality and Safety

- Server-side validation for all writes
- Consistent error format: { error: { code, message, details? } }
- Avoid silent failures; surface actionable messages in UI
- Basic rate limiting optional (nice-to-have)

## Data Model

### Entities

#### Todo

- id (string, uuid)
- title (string, 1..200)
- description (string, optional, 0..2000)
- status ("active" | "completed")
- due_date (ISO string, optional)
- created_at (ISO string)
- updated_at (ISO string)
- completed_at (ISO string, optional)
- list_id (string, optional; if lists enabled)

#### List

- id (string, uuid)
- name (string, 1..80)
- created_at (ISO string)
- updated_at (ISO string)

#### Tag

- id (string, uuid) OR use string tag names only (simpler)
- name (string, 1..40)

#### Todo_Tag

- todo_id
- tag_id OR tag_name

## Database Schema

### Tables

#### todos

- id TEXT PRIMARY KEY
- title TEXT NOT NULL
- description TEXT
- status TEXT NOT NULL CHECK(status IN ('active','completed'))
- due_date TEXT
- created_at TEXT NOT NULL
- updated_at TEXT NOT NULL
- completed_at TEXT
- list_id TEXT

#### lists

- id TEXT PRIMARY KEY
- name TEXT NOT NULL
- created_at TEXT NOT NULL
- updated_at TEXT NOT NULL

#### tags

- id TEXT PRIMARY KEY
- name TEXT NOT NULL UNIQUE

#### todo_tags

- todo_id TEXT NOT NULL
- tag_id TEXT NOT NULL
- PRIMARY KEY (todo_id, tag_id)

### Indexes

- CREATE INDEX idx_todos_status ON todos(status)
- CREATE INDEX idx_todos_due_date ON todos(due_date)
- CREATE INDEX idx_todos_updated_at ON todos(updated_at)

## API Endpoints Summary

### Health

- GET /api/health

### Todos

- GET /api/todos
    - Query params: status=all|active|completed, q=search, sort=created_at|due_date|title, order=asc|desc, listId?
- POST /api/todos
    - Body: { title, description?, dueDate?, listId?, tags? }
- GET /api/todos/:id
- PUT /api/todos/:id
    - Body: { title?, description?, dueDate?, status?, listId?, tags? }
- DELETE /api/todos/:id
- POST /api/todos/:id/toggle
    - Body: { completed: boolean }
- POST /api/todos/clear-completed

### Lists

- GET /api/lists
- POST /api/lists
- PUT /api/lists/:id
- DELETE /api/lists/:id

### Tags (Optional)

- GET /api/tags

## UI Layout

### Main Structure

- Header: app title, list selector (optional), search input
- Main: todo input row + todo list
- Footer: items left, filter pills (All/Active/Completed), "Clear completed"
- Optional right-side panel or modal: todo details editing

### Components

#### Todo Input

- Single-line input for title; optional expand for description
- Add button + Enter key submit
- Disabled while saving

#### Todo Item

- Checkbox for complete
- Title + optional due date badge
- Edit action (inline)
- Delete action with confirmation

#### Filters

- All / Active / Completed
- Shows counts and current selection

## Implementation Steps

### Step 1: Backend Foundation

- Initialize monorepo with Bun and Spernakit template
- Set up /frontend and /backend workspaces
- Configure bunfig.toml and workspace scripts
- Configure ESLint, Prettier, and TypeScript strict mode
- Create config/todolist.json and environment-specific variants
- Initialize Prisma with SQLite database and base schema
- Implement todos CRUD with validation (Zod)
- Add error handler + consistent error response
- Create health endpoint

### Step 2: Frontend Foundation

- Create API client + typed DTOs
- Build base layout and todo list rendering
- Implement create/toggle/delete flows
- Add loading/error states + optimistic updates
- Implement edit flow, filters, sorting, and search
- Add accessibility and keyboard shortcuts

### Step 3: Polish UX and Correctness

- Add tests for key API routes and core UI behaviors
- Optimize for mobile responsiveness
- Add keyboard navigation
- Implement inline editing
- Add confirmation dialogs for destructive actions
- Polish animations and transitions

## Success Criteria

### Functionality

- All todo CRUD operations work reliably against the backend
- Filters/search/sort behave correctly and remain consistent with server state
- Data persists across page reloads

### User Experience

- UI is responsive and keyboard accessible
- Clear feedback for loading, success, and error conditions
- Interactions feel fast (optimistic UI where appropriate)

### Technical Quality

- Type-safe DTOs between frontend and backend
- Request validation on backend for all write endpoints
- Tests cover core flows (at least: create, toggle, edit, delete, filter)

## Spernakit-Specific Optimizations

### Performance

- Virtual scrolling for large todo lists (if needed)
- Strategic DB indexing for status and date queries
- Code splitting and lazy loading in Vite
- Bundle optimization and tree-shaking

### Security

- Input validation with Zod schemas
- Rate limiting on API endpoints
- CORS configuration for development
- XSS protection in todo rendering

### Development Workflow

- Single set of Bun scripts for lint/format/build/test
- Shared TypeScript types for API contracts
- Standardized error response format
- Automated migrations as part of deploy pipeline
- JSON-based configuration for all environments

## Integration Dependencies

### Third-party Libraries

- React 19 with Vite 5
- Tailwind CSS with DaisyUI
- TanStack Query
- Prisma ORM
- Zod (validation)
- bcrypt (password hashing, if auth added)
- jsonwebtoken (JWT management, if auth added)

### API Integrations

- Native Node fs module (if file attachments added)
- WebSockets (if real-time collaboration added)

## Deployment Strategy

- Monolithic Docker container with nginx + supervisord
- Frontend port (3330) exposed externally
- Backend port (3331) internal only
- Health and metrics endpoints wired into orchestrator probes
- Persistent volume mounts for database and configuration
- Environment-specific config via JSON files
