# Draft: TaskFlow - Task Manager App

## Requirements (confirmed)
- **Technology Stack**: .NET 8.0 Core + Blazor Server
- **Project Name**: TaskFlow (based on PRD)
- **Architecture**: Clean Architecture pattern (Domain, Application, Infrastructure, Web)
- **Database**: PostgreSQL (or SQLite for dev) - TBD
- **Auth**: ASP.NET Identity with role-based authorization (Owner, Member, Viewer)
- **Real-time**: SignalR for comments and board sync
- **Styling**: Tailwind CSS + CSS Isolation
- **JS Interop**: SortableJS for drag & drop Kanban
- **Email**: MailKit + MimeKit
- **File Storage**: Local disk / Azure Blob (configurable)

## Current State
- Fresh .NET 8 Blazor Server project with default template
- Only basic Razor Components structure exists
- No database, auth, EF Core, SignalR, or additional packages installed
- Located at: E:\CODE\LearnBlazor

## Decisions Confirmed
1. **Scope**: Week 1 Milestone — Foundation & Auth only (Project setup, Identity, layout, DB migration)
2. **Database**: PostgreSQL (full setup with EF Core)
3. **Testing**: No tests for now
4. **File Storage**: Local disk (wwwroot/uploads)
5. **Goal**: Production-ready reference project for learning Blazor
6. **Email**: Console/file logging for dev (MailKit configured but SMTP mock)
7. **Styling**: Tailwind CSS + CSS Isolation

## Scope Boundaries (Week 1)
- **INCLUDE**: 
  - Clean Architecture folder structure (Domain, Application, Infrastructure, Web)
  - PostgreSQL + EF Core setup with initial migration
  - ASP.NET Identity with role-based auth (Owner, Member, Viewer)
  - Tailwind CSS integration
  - Base layout, navigation, landing page
  - Login/Register pages with DataAnnotations validation
  - Health check endpoint
  - Middleware pipeline configuration
- **EXCLUDE**: 
  - Project/Board/Card features (Week 2+)
  - SignalR (Week 3)
  - File upload UI (Week 4)
  - Email notifications worker (Week 4)
  - Drag & drop (Week 3)
  - Dashboard analytics (Week 4)
