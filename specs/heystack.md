# Heystack - File Analysis Platform on Spernakit

**Version:** 1.0.0
**Template:** Spernakit v1.7.0
**Stack:** SPERN (SQLite, Prisma, Express, React, Node.js on Bun)

## Overview

Heystack is a sophisticated file analysis and monitoring web application that provides intelligent file system monitoring, content analysis, and rich visual insights. This specification describes a Spernakit-based implementation of Heystack that preserves the existing product goals while replatforming onto the SPERN stack with JSON-driven configuration.

The application must:

- Continuously monitor configured file system paths with recursive traversal and metadata tracking
- Detect and record file changes (new, modified, deleted, or quarantined) with detailed scan history
- Perform advanced content analysis (text extraction, summarization, categorization, tagging, duplicate detection) at scale
- Provide powerful search, filtering, and discovery across files, summaries, categories, tags, and scan results
- Visualize file relationships, activity timelines, category distributions, and tag clouds using D3-powered dashboards
- Offer configurable automation and scheduling for scans with resource management, progress tracking, and retry behavior
- Support compliance and auditing use cases through comprehensive history, alerting, and exportable reports

The target audience includes system administrators, content managers, data analysts, file system auditors, and knowledge management teams who need deep visibility into large file systems. The UI presents a modern, responsive dashboard experience (dark/light modes, Flowbite/DaisyUI patterns) while the backend provides secure, observable, and performant services aligned with Spernakit architecture.

## Technology Stack

### Spernakit Architecture

- **Core Stack**: SPERN (SQLite, Prisma, Express, React, Node.js)
- **Runtime**: Bun for both frontend and backend services
- **Package Manager**: Bun for dependency management and scripts
- **Monorepo Structure**: Workspace architecture with frontend/backend separation
- **Configuration**: JSON-driven config via `config/heystack.json` (no implicit .env loading)

### Frontend

- **Framework**: React 19 with Vite 5
- **Styling**: Tailwind CSS with DaisyUI component library
- **State Management**:
    - TanStack Query (React Query) for server state
    - React Context + hooks for app-level state (auth, theme, layout, project)
- **Routing**: React Router with ProtectedRoute components for RBAC-aware routing
- **Markdown**: React Markdown (and plugins) for message + artifact rendering
- **Code Highlighting**: Syntax highlighting for code blocks using a Monaco-compatible theme with copy buttons and optional language selector
- **UI Components**: Primary: DaisyUI components; Secondary patterns: Flowbite-inspired cards, dashboards, and status widgets adapted to DaisyUI themes and Spernakit style rules
- **Visualization**: D3.js v7 for advanced data visualization (file relationship graphs, activity timelines, category/tag distributions, search/usage analytics)
- **File Insights UI**: Heystack-style panels integrated into the main layout (scan activity stream, progress indicators, tag clouds, category breakdowns, relationship graphs, drill-down detail views)
- **Progressive Enhancements**: HTMX/\_hyperscript-inspired micro-interactions implemented as React hooks and lightweight helper utilities (scheduler controls, inline filters, quick actions on lists, optimistic UI updates)
- **Port**: {frontend_port} (default: 3330)

### Backend

- **Runtime**: Node.js with Express 5, running on Bun
- **Database**: SQLite with Prisma ORM (PostgreSQL-compatible schema)
- **ORM**: Prisma ORM with automated migrations, type-safe client, and Studio
- **API Integration**: Claude API via Anthropic SDK for file/content analysis (summarization, tagging, explanation) and tools that operate over Heystack scan results
- **Streaming**:
    - Server-Sent Events (SSE) for streaming assistant responses related to file analysis and investigations
    - WebSocket channels for notifications, scan progress, and dashboards
- **Authentication**: JWT with HTTP-only cookies and a 5-tier RBAC system (SYSOP > ADMIN > MANAGER > OPERATOR > VIEWER with hierarchical inheritance)
- **Security**: CSP headers, CORS allowlists per environment, rate limiting (per-IP and per-endpoint tiers), input validation with Joi or Zod schemas, comprehensive audit logging
- **File Scanner Service**: Node worker (or background service) with optional PowerShell child-process bridge that reuses Heystack scanner modules (recursive traversal, change detection, checksum generation, binary vs text detection, progress reporting, partial scan resumption, scale target: 1M+ files)
- **Content Analysis Pipeline**: Background jobs orchestrated via scheduler queues (text extraction, summarization, categorization, dynamic tag generation, file relationship detection, duplicate detection)
- **Scheduler Engine**: Cron-expression and calendar-based schedules, timezone-aware execution, pause/resume/cancel operations, resource throttling, scan queues, retry with exponential backoff
- **Observability**: Structured JSON logging with correlation IDs, health/readiness/liveness endpoints, Prometheus-friendly metrics, error-rate and latency tracking, WebSocket stream for real-time activity

### Configuration

All configuration stored in `config/heystack.json`:

- Server ports (frontend: 3330, backend: 3331)
- Database path: `file:../../data/heystack.db`
- Claude API key path reference (e.g., `/tmp/api-key`)
- Security keys and secrets
- File scanner paths and patterns
- Scheduler settings and resource limits
- Feature flags and environment settings

## Prerequisites

### Environment Setup

- Application configuration stored in `config/heystack.json`
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

### Assistant Panel

- Heystack assistant panel for asking questions about scanned files, changes, and tags, using a clean, centered layout with familiar message bubbles
- Streaming responses with typing indicators and partial message rendering
- Markdown rendering (lists, headers, tables) and inline images
- Code blocks with syntax highlighting, language labels, and copy button
- LaTeX/math equation rendering for technical content
- Image upload and inline preview in messages (with audit trail)
- Multi-turn conversational context with correct model parameters
- Message editing and regeneration (per-message) with history tracking
- Stop-generation button during streaming
- Auto-resizing textarea with character count and token estimation
- Keyboard shortcuts (Enter to send, Shift+Enter for newline)

### Artifacts

- Automatic detection of artifact-style responses (code, diagrams, docs)
- Side-panel artifact viewer with tabbed layout
- Code artifact viewer with diffing between versions
- HTML/SVG live preview
- React component preview (where safe/practical)
- Mermaid diagram rendering
- Text document artifact support with full-screen mode
- Artifact editing and re-prompting based on current version
- Download/copy artifact content
- Artifact versioning and history with audit log entries

### Investigation Management

- Create, rename, duplicate, and delete (soft-delete) investigation sessions over Heystack file and scan data
- Sidebar list of investigation sessions with virtual scrolling for large histories
- Automatic title generation from first exchange (with inline rename)
- Investigation search by title/content and related file context
- Pin and archive investigation sessions
- Folder- and project-based organization
- Export investigation history (assistant prompts/responses plus key file references) to JSON, Markdown, and PDF
- Timestamps (created, updated, last accessed)
- Unread indicators and basic activity markers
- Full audit trail of investigation-level actions

### Projects

- Create projects to group monitored paths, scan configurations, investigation sessions, artifacts, and knowledge base docs
- Upload project knowledge base documents with metadata
- Project-specific custom instructions and defaults
- RBAC-based project sharing (mock or real, depending on deployment)
- Project analytics (usage, scan coverage, key tags)
- Project templates for common use cases

### Model Selection

- Model selector with at least these options (names illustrative):
    - Claude Sonnet 4.5 (default)
    - Claude Haiku 4.5
    - Claude Opus 4.1
- Context window and capability indicators per model
- Model pricing information (display only, no billing integration)
- Ability to switch models mid-conversation with clear UX
- Optional model comparison/help view

### Custom Instructions

- Global custom instructions per user
- Project-specific instructions layered on global
- Conversation-specific system prompts layered on project/global
- Instruction templates and presets
- Preview area explaining how instructions will influence responses

### Settings & Preferences

- Theme selection (Light, Dark, Auto/system)
- Font size and message density controls
- Code theme selection
- Advanced toggles (show tokens, developer-mode panes, experimental features)

### File Monitoring

- Configurable root paths and include/exclude patterns
- Recursive directory traversal with metadata capture
- Change detection (new/modified/deleted files) with snapshots
- File type identification (by extension and magic bytes)
- File metadata extraction (size, timestamps, permissions where available)
- Binary-safe scanning and text extraction
- Efficient handling of 1M+ files with indexing and pagination
- Real-time change notifications surfaced into UI via WebSocket

### Content Analysis

- Text extraction across common document formats
- Summarization and key-phrase extraction via Claude
- Categorization with confidence scores
- Automatic tag generation based on content and context
- Duplicate and near-duplicate detection
- Relationship mapping (files-to-files, files-to-conversations)

### Scheduling

- Cron-like and calendar-style scheduling for scans and automations
- Support for presets (hourly, daily, weekly, monthly) and custom cron strings
- Timezone-aware scheduling with DST-safe logic
- Pause/resume/cancel operations with audit logs
- Resource throttling knobs (concurrency, IO caps)
- Retry with exponential backoff and capped attempts
- History view of schedule runs with metrics and outcomes

### Visualization Dashboard

- D3-based dashboards for scan coverage, tag distributions, trends
- Relationship graphs linking files, tags, and conversations
- Zoom/pan/hover interactions with performance targets (smooth at 60fps)
- Export visuals to PNG/SVG/PDF formats
- Configurable layouts (panels, saved views)

### Admin, Audit & Security

- RBAC-aware admin views (SYSOP/ADMIN only) for user and role management, system metrics, audit log browsing
- Comprehensive audit logging for all sensitive actions
- Soft-delete patterns for key entities

## Key Interactions

### Assistant Investigation Flow

1. User selects or creates a project (optional but recommended)
2. User starts a new investigation session or resumes an existing one
3. User configures model and key parameters (temperature, tools) as needed
4. User types a prompt and sends
5. Backend streams assistant responses via SSE, UI renders progressively
6. Any detected artifacts appear in the side panel with preview
7. User may edit messages, regenerate responses, or branch the investigation session
8. All actions are logged for auditing

### File Scan Flow

1. Admin or authorized user configures root paths and filters
2. User defines one-off scan or schedule using scheduler UI
3. Scan is queued and executed by scanner service with progress updates
4. UI displays progress (files processed, estimated time, anomalies)
5. Results (files, tags, relationships) feed into dashboards and search
6. Subsequent scans re-use checksums and change tracking for efficiency

### Artifact Flow

1. Claude response contains a code block or structured content
2. Application detects artifact and surfaces it in side panel
3. User opens artifact, views, and optionally edits it
4. User can re-prompt Claude using artifact content as context
5. User may download, copy, or full-screen the artifact
6. Version history and relevant audit events are recorded

### Investigation Management Flow

1. User clicks "New Investigation" or selects a saved investigation session
2. Investigation auto-saves after the first assistant interaction
3. Titles auto-generate and may be edited inline
4. Investigation sessions can be pinned, archived, or moved into folders/projects
5. Search and filters help locate investigation sessions quickly
6. Actions update audit logs and respect RBAC

## Implementation Steps

### Step 1: Project Foundation

- Initialize monorepo with Bun and Spernakit template
- Set up /frontend and /backend workspaces
- Configure bunfig.toml and workspace scripts
- Configure ESLint, Prettier, and TypeScript strict mode
- Create config/heystack.json and environment-specific variants
- Initialize Prisma with SQLite database and base schema
- Seed default RBAC roles and test users

### Step 2: Authentication, RBAC, and Security

- Implement JWT-based auth with HTTP-only cookies
- Implement 5-tier RBAC system with inheritance
- Add ProtectedRoute components and RBAC-aware menus
- Implement authorize() middleware in backend
- Set up baseline audit logging service
- Configure CORS, CSP, rate limiting, and validation

### Step 3: Core Database Schema

- Model users, roles, projects, investigation sessions, messages, and artifacts
- Model file entities, tags, relationships, and scan metadata
- Model schedules, jobs, and job runs for the scheduler
- Add audit fields and soft-delete patterns where required
- Add strategic indexes for common queries
- Generate migrations and run them via Bun scripts

### Step 4: Assistant Panel and Analysis Integration

- Build assistant panel layout with sidebar + main panel for investigations over Heystack data
- Implement SSE-based streaming response handling
- Integrate Anthropic SDK behind backend proxy routes
- Add markdown + code block rendering on frontend
- Implement stop-generation, regenerate, and edit flows
- Add model selection and advanced settings UI

### Step 5: Investigation Sessions and Projects

- Implement CRUD for investigation sessions and folders/projects
- Add sidebar with virtual scrolling and search
- Implement pin, archive, and soft-delete
- Implement project selector and project-scoped views
- Add export functionality (JSON, Markdown, PDF skeleton)

### Step 6: Artifacts System

- Detect artifact-worthy content in Claude responses
- Implement artifact panel with tabs and previews
- Add versioning and editing flows
- Integrate Monaco-style code viewer and Mermaid renderer
- Hook artifact actions into audit logging

### Step 7: Heystack File Scanner and Content Analysis

- Implement scanner service and filesystem adapters
- Integrate optional PowerShell bridge for Windows
- Model scans, scan runs, and file metadata in Prisma
- Implement content extraction and analysis pipeline
- Store summaries, tags, relationships, and checksums
- Ensure scanner performance and resilience (retries, resumption)

### Step 8: Scheduler and Automation

- Implement scheduler engine with cron and calendar rules
- Persist schedule definitions and runs
- Expose scheduler APIs and admin UI
- Wire scheduler to scanner and other automations
- Add pause/resume/cancel and resource throttling controls

### Step 9: Dashboards and Visualizations

- Build D3-based dashboards for key metrics
- Implement relationship graphs with zoom/pan/hover
- Add export of charts/graphs
- Optimize visualizations for performance

### Step 10: Security, Observability, and Admin UX

- Finalize audit logging and admin dashboards
- Implement health, readiness, and liveness endpoints
- Expose metrics endpoint for Prometheus
- Add admin-only views for logs, metrics, and scans
- Harden error handling and user-facing error messages

### Step 11: Polish, Testing, and Deployment

- Implement full testing strategy (unit, integration, E2E, performance)
- Polish UI for accessibility (WCAG 2.1 AA) and responsiveness
- Optimize bundle sizes and load times
- Build Docker image with nginx + supervisord
- Configure production deployment, backups, and monitoring

## Success Criteria

### Functionality

- Smooth SSE streaming for Heystack assistant responses related to file analysis, with a responsive UI
- Accurate artifact detection and rendering
- Reliable conversation/project management with soft delete
- Correct RBAC enforcement across all key features
- File scanning and analysis functioning at target scale
- Scheduling system reliably executing jobs with retries
- Dashboards and visualizations updating based on live data

### User Experience

- UI uses a familiar assistant-style layout inspired by modern tools while respecting Spernakit style and emphasizing file analysis workflows
- Responsive layout across desktop, tablet, and mobile
- Clear feedback for all user actions (loading, errors, confirmations)
- Fast perceived performance, minimal visible blocking
- Keyboard-accessible navigation and controls

### Technical Quality

- TypeScript strict mode passes with no errors
- Lint and format checks pass with no violations
- bun run build succeeds without warnings that indicate broken behavior
- Database schema and queries validated under load
- Error handling and logging present at all critical paths

### Design Polish

- Consistent typography, spacing, and component usage
- Fully functional dark mode with DaisyUI themes
- Smooth micro-interactions and transitions
- Accessible color contrast and focus states

### Security & Compliance

- RBAC hierarchy enforced on backend and reflected in UI
- All sensitive actions logged with sufficient context
- Input validation and sanitization for all external inputs
- Rate limiting active and observable on sensitive routes
- Authentication implemented with HTTP-only cookies and secure defaults
- Soft delete patterns applied where required

### File Management

- Scanning handles 1M+ files with acceptable performance
- Change detection accurately tracks new/modified/deleted files
- Checksums generated and stored for integrity and deduplication
- Text extraction and categorization produce actionable results
- Duplicate and relationship views behave correctly in UI

### Scheduling Reliability

- Cron expressions parsed and executed as expected
- Timezone handling validated over DST transitions
- Resource limits respected under high load
- Retry and backoff strategies verified via tests
- Run history and metrics visible for operators

### Visualization Quality

- Charts and graphs render in <= 200ms for 10,000 items
- Graph layouts complete in <= 1s for ~1,000 nodes
- Interactions (zoom/pan/hover) remain smooth (~60fps)
- Exported images/vectors are sharp and legible

### Performance Benchmarks

- API: 95% requests <= 500ms; p99 within acceptable SLO
- DB: 95% complex queries <= 100ms
- UI: initial page usable in <= 200ms on modern hardware
- Search: results for 1M files <= 500ms (p95)
- Concurrent users: stable for 50+ concurrent sessions

## Spernakit-Specific Optimizations

### Performance

- Virtual scrolling for large lists (10,000+ conversations/files)
- Strategic DB indexing for hot paths
- Code splitting and lazy loading in Vite
- Gzip/Brotli compression for text responses
- Bundle optimization and tree-shaking
- In-memory caching for frequently accessed data

### File Management

- Incremental scanning with changed-only diffs
- Efficient B-tree indexing on key file metadata fields
- Resource-aware scanning with CPU/memory/IO caps
- Background jobs for heavy content analysis

### Observability

- Structured JSON logs with correlation IDs
- Health, ready, and live endpoints for container orchestration
- Metrics endpoint compatible with Prometheus
- Real-time monitoring dashboards for admins
- Threshold-based alerting for error rates and slowdowns

### Security

- 5-tier RBAC with inheritance and clear role definitions
- Comprehensive audit trail covering core entities and actions
- Input validation with shared schema definitions
- CSP, CSRF protection, secure cookies
- Rate limiting with per-IP and per-endpoint rules

### Development Workflow

- Single set of Bun scripts for lint/format/build/test
- Shared TypeScript types for API contracts
- Standardized error response format
- Automated migrations as part of deploy pipeline
- JSON-based configuration for all environments

## Integration Dependencies

### Third-party Libraries

- D3.js v7
- TanStack Query
- React Router
- DaisyUI + Tailwind CSS
- Prisma ORM
- Anthropic SDK
- Joi/Zod (validation)
- bcrypt (password hashing)
- jsonwebtoken (JWT management)

### API Integrations

- Claude API via backend proxy
- Native Node fs module for file operations
- WebSockets for notifications and scan progress
- SSE for streaming assistant/file-analysis responses

## Deployment Strategy

- Monolithic Docker container with nginx + supervisord
- Frontend port (3330) exposed externally
- Backend port (3331) internal only
- Health and metrics endpoints wired into orchestrator probes
- Persistent volume mounts for database and configuration
- Environment-specific config via JSON files
