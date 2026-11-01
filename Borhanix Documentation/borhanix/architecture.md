# Architecture

## Overview
Borhanix is a monolithic **ASP.NET Core MVC** application following a layered architecture: Controllers → Services → Data (EF Core) → SQL Server, with Razor Views rendering the UI.

```
┌─────────────────────────────────────────────────────────┐
│                        Browser                          │
│        (Razor Views + Tailwind CSS + JavaScript)        │
└───────────────────────────┬───────────────────────────────┘
                            │ HTTP
┌───────────────────────────▼───────────────────────────────┐
│                       Controllers                         │
│   (Auth, Course, Dashboard, Instructor, Admin, Quiz, …)   │
└───────────────────────────┬───────────────────────────────┘
                            │
┌───────────────────────────▼───────────────────────────────┐
│                        Services                           │
│  (business logic: enrollment, attendance, AI assistant,   │
│   certificates, progress tracking)                        │
└───────────────────────────┬───────────────────────────────┘
                            │
┌───────────────────────────▼───────────────────────────────┐
│                    Data (EF Core DbContext)                │
│                     + Migrations                           │
└───────────────────────────┬───────────────────────────────┘
                            │
┌───────────────────────────▼───────────────────────────────┐
│                        SQL Server                          │
└─────────────────────────────────────────────────────────────┘
```

## Layers

### Controllers
Handle incoming requests, apply `[Authorize]`/role checks, call into Services or the DbContext directly for simpler reads, and return Views or ViewModels. Controllers are organized by feature area (e.g., Course, Instructor, Admin, Account/Auth, Quiz/Assignment, Attendance, AI Assistant).

### Services
Encapsulate business logic that shouldn't live in Controllers, such as:
- Enrollment approval workflows
- Attendance processing (QR code validation)
- Certificate generation
- AI assistant orchestration (calls to DeepSeek via OpenRouter)
- Course duplication (`DuplicateCourse`) — a deep-copy operation for cloning course structure and content

### Data Layer
- **DbContext** — central EF Core context mapping domain Models to SQL Server tables.
- **Migrations** — version-controlled schema changes, applied via `dotnet ef database update`.

### Models & ViewModels
- **Models** represent persisted domain entities (Users, Courses, Enrollments, Lessons, Quizzes, Assignments, Attendance, Certificates).
- **ViewModels** shape data for specific Views (e.g., combining a course with enrollment/progress data for a dashboard view) and carry validation attributes for form binding.

### Views
Razor views styled with Tailwind CSS v4, following a consistent design system:
- Brand color `#006FFF` mapped to `bg-primary`/`text-primary` tokens
- `rounded-2xl`/`rounded-3xl` with `slate`/`gray` palettes (Notion/Linear/Stripe-inspired aesthetic)
- Dark mode via a `.dark` class strategy with explicit `dark:` overrides on layered CSS tokens (`--bg`, `--surface`, `--surface-2`, `--color-third`)
- Lucide icons via `data-lucide` attributes, rendered client-side
- `<template>` elements used for the lesson player, cloned client-side on interaction rather than re-rendered server-side

## Authentication & Authorization
ASP.NET Identity provides cookie-based authentication and role-based authorization for three roles: **Student**, **Instructor**, and **Admin**. Role checks are applied via `[Authorize(Roles = "...")]` on controllers/actions, and role-aware UI is rendered per dashboard.

## Frontend Build Pipeline
Tailwind CSS v4 is integrated into the MSBuild pipeline via the Tailwind CLI, supporting `--watch` mode during development so utility classes are regenerated automatically as Razor views change. `tailwind.extension.json` defines the project's design tokens (brand colors, surfaces, radii).

## AI Assistant Integration
The AI assistant feature calls out to the DeepSeek model through the OpenRouter API. This integration is isolated in a dedicated Service so the API key and request/response handling are centralized and not scattered across controllers.

## Design Decisions
- **Server-rendered MVC over SPA**: Razor Views were chosen over a separate frontend (the project originally prototyped a Next.js/React frontend before migrating to ASP.NET Razor Views) to keep the stack simpler and reduce API surface area for a small team.
- **Service layer for cross-cutting logic**: Business rules like attendance validation and certificate issuance live in Services, not Controllers, to keep them testable and reusable.
- **Template-based client interactivity**: `<template>` cloning is preferred over full page reloads for the lesson player to keep navigation fast without introducing a SPA framework.

## Related Documents
- [Database Design](database.md)
- [API Documentation](api.md)
- [Deployment](deployment.md)
