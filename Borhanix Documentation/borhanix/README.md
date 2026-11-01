# Borhanix

## Overview
Borhanix is a modern, role-based **Learning Management System (LMS)** built to help learners develop real-world skills through structured, high-quality courses. It supports instructors creating and managing course content, students learning and tracking progress, and administrators overseeing the platform — all in a single ASP.NET Core MVC application.

Internally the solution is named `DepiLms` (see `DepiLms.csproj` / `DepiLms.slnx`), while the product is branded and marketed as **Borhanix**.

> Repository: https://github.com/kh1aled/borhanix

## Objectives
- Provide a structured, self-hosted platform for delivering and consuming online courses.
- Give instructors full control over course creation, content management, and student progress.
- Give students a clear, distraction-free learning experience with progress tracking and certification.
- Support in-person/hybrid learning through QR-based attendance tracking.
- Offer an AI-assisted learning companion to help students inside the platform.
- Keep the codebase framework-idiomatic (ASP.NET Core MVC conventions) so it stays maintainable and easy to extend toward a SaaS/multi-tenant product.

## Features
- 🔐 **Authentication & Authorization** — ASP.NET Identity with role-based access (Student, Instructor, Admin)
- 👨‍🏫 **Instructor Studio** — course creation, content management (accordion-based content sidebar), course duplication
- 👨‍🎓 **Student Dashboard** — enrolled courses, progress, certificates
- 🛠 **Admin Dashboard** — user management, enrollment approvals, platform oversight
- 📚 **Course Catalog & Course Details** — browsing and discovery of available courses
- 🎥 **Learn Page / Lesson Player** — structured lesson consumption with progress tracking
- 📝 **Quiz & Assignment Engine** — course assessments tied to student progress
- 🧾 **Certificates** — certificate generation/display on course completion
- 📶 **QR-Based Attendance Tracking** — QR scanner + student ID card flow for session attendance
- 🤖 **AI Assistant** — DeepSeek-powered chat assistant (via OpenRouter) to support learners
- 🌗 **Dark Mode** — class-based dark theme across the UI
- 🔔 **Toast Notifications** — user feedback via Toastify.js

## Tech Stack

### Backend
- ASP.NET Core MVC (C#)
- Entity Framework Core
- ASP.NET Identity (authentication & authorization)
- LINQ

### Frontend
- Razor Views
- Tailwind CSS v4 (Tailwind CLI integrated with MSBuild, `--watch` mode)
- JavaScript
- Lucide Icons (`data-lucide`)
- Toastify.js

### Database
- SQL Server (via Entity Framework Core migrations)

### AI Integration
- DeepSeek model via OpenRouter API

## System Requirements
- [.NET SDK](https://dotnet.microsoft.com/download) 8.0 or later
- [Node.js](https://nodejs.org/) 18+ and npm (for Tailwind CSS build tooling)
- SQL Server (LocalDB, Developer, or full Server edition) or a compatible SQL Server instance
- A code editor / IDE such as Visual Studio 2022 or VS Code
- Git

## Installation

### Prerequisites
Make sure the following are installed before setting up the project:
- .NET SDK 8.0+
- Node.js 18+ and npm
- SQL Server (LocalDB is sufficient for local development)
- Git

### Clone the Repository
```bash
git clone https://github.com/kh1aled/borhanix.git
cd borhanix
```

### Install Dependencies
Restore .NET packages:
```bash
dotnet restore
```

Install Node/Tailwind dependencies:
```bash
npm install
```

### Configure Environment Variables
Update `appsettings.json` / `appsettings.Development.json` with your local configuration, including:
- `ConnectionStrings:DefaultConnection` — SQL Server connection string
- Identity/authentication settings (if applicable)
- AI assistant configuration — OpenRouter API key used for the DeepSeek assistant

Example connection string:
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=BorhanixDb;Trusted_Connection=True;MultipleActiveResultSets=true"
  }
}
```

> Never commit real secrets to source control — for local development, prefer [.NET User Secrets](https://learn.microsoft.com/aspnet/core/security/app-secrets) for API keys and connection strings.

### Database Setup
Apply Entity Framework Core migrations to create the database schema:
```bash
dotnet ef database update
```

If you need to add a new migration after modifying models:
```bash
dotnet ef migrations add <MigrationName>
```

### Run the Project
Build and run the Tailwind CSS watcher alongside the app (or as part of the MSBuild pipeline), then start the application:
```bash
dotnet run
```

By default the app will be available at `https://localhost:5001` (or the port configured in `Properties/launchSettings.json`).

## Project Structure
```
borhanix/
│
├── Controllers/         # MVC controllers (routing & request handling)
├── Data/                 # DbContext and data access configuration
├── Migrations/            # EF Core database migrations
├── Models/                # Domain/entity models
├── Properties/             # Launch settings and project properties
├── Services/                # Application/business logic services
├── ViewModels/               # View-specific data transfer models
├── Views/                     # Razor views (UI)
├── wwwroot/                    # Static assets (CSS, JS, images, compiled Tailwind output)
├── docs/                        # Project documentation (this folder)
├── DepiLms.csproj                # Project file
├── Program.cs                     # Application entry point & startup configuration
├── appsettings.json                # App configuration
├── package.json                     # Node/Tailwind tooling dependencies
├── tailwind.extension.json           # Tailwind design token extensions
└── README.md
```

## Architecture
Borhanix follows the standard **ASP.NET Core MVC** architectural pattern with a service layer for business logic:

- **Controllers** handle incoming HTTP requests, coordinate with Services and the Data layer, and select Views/ViewModels to return.
- **Services** encapsulate business logic (e.g., course enrollment, attendance processing, AI assistant integration) so Controllers stay thin.
- **Data (DbContext)** manages persistence via Entity Framework Core, with schema evolution handled through Migrations.
- **Models** represent core domain entities (Users, Courses, Enrollments, Quizzes, Assignments, Attendance, Certificates).
- **ViewModels** shape and validate data specifically for Views, decoupling the UI from domain models.
- **Views** are Razor views styled with Tailwind CSS v4, using ASP.NET tag helpers (`asp-for`, `asp-action`, `asp-route-*`) and partials/templates for reusable UI (e.g., `<template>` elements for the lesson player).

See [docs/architecture.md](docs/architecture.md) for a deeper breakdown.

## Database Design

### Entity Relationship Diagram (ERD)
See [docs/diagrams/erd.png](docs/diagrams/erd.png) and [docs/database.md](docs/database.md) for the full schema.

### Database Tables
Core tables are generated from the `Models/` entities via EF Core migrations and typically include:
- Users / Identity tables (roles: Student, Instructor, Admin)
- Courses
- Enrollments
- Lessons / Course Content
- Quizzes & Assignments
- Attendance Records
- Certificates

Full column-level detail is documented in [docs/database.md](docs/database.md).

## User Roles & Permissions
| Role | Description | Key Permissions |
|------|-------------|------------------|
| **Student** | Learner using the platform | Browse catalog, enroll in courses, view lessons, take quizzes/assignments, track progress, view certificates, mark attendance via QR |
| **Instructor** | Course creator/owner | Create/manage courses, manage content (Instructor Studio), duplicate courses, review student progress, manage attendance sessions |
| **Admin** | Platform administrator | Manage users, approve enrollments, oversee all courses, access admin dashboard and platform-wide controls |

## Application Workflow
1. A user registers/logs in and is assigned a role (Student by default; Instructor/Admin assigned by an admin).
2. **Students** browse the course catalog, view course details, and enroll.
3. **Instructors** build courses in the Instructor Studio, adding lessons/content and assessments.
4. Enrolled students progress through lessons via the Learn page, complete quizzes/assignments, and track their progress.
5. Attendance for live/in-person sessions is tracked via QR code scanning against a student's ID card.
6. On completion, students receive a certificate.
7. Admins approve enrollments and manage users/courses from the Admin Dashboard.
8. Students can use the AI assistant at any point for help related to their learning.

## Screenshots

### Home Page
_Add screenshot: `docs/screenshots/home.png`_

### Dashboard
_Add screenshot: `docs/screenshots/dashboard.png`_

### Login
_Add screenshot: `docs/screenshots/login.png`_

### Other Pages
_Add additional screenshots to `docs/screenshots/`_

## API Documentation
See [docs/api.md](docs/api.md) for full details, including authentication, endpoints, and request/response examples.

### Authentication
Authentication is handled via ASP.NET Identity (cookie-based for the MVC app). See [docs/api.md](docs/api.md#authentication).

### Endpoints
Controller-based endpoints are documented per-controller in [docs/api.md](docs/api.md#endpoints).

### Request & Response Examples
See [docs/api.md](docs/api.md#request--response-examples).

## Configuration
Key configuration lives in:
- `appsettings.json` / `appsettings.Development.json` — connection strings, environment-specific settings
- `tailwind.extension.json` — Tailwind design tokens (brand color `#006FFF`, layered surface tokens, dark mode overrides)
- `package.json` — Tailwind CLI build scripts
- `.NET User Secrets` (recommended) — API keys such as the OpenRouter key for the AI assistant

## Security
- Authentication and authorization enforced via ASP.NET Identity with role-based access control.
- Sensitive configuration (connection strings, API keys) should be stored via User Secrets/environment variables, not committed to source control.
- Anti-forgery tokens are used on state-changing forms via ASP.NET tag helpers.
- Role checks should be enforced both at the controller level (`[Authorize(Roles = "...")]`) and reflected in the UI.

See [docs/api.md](docs/api.md#authentication) and [docs/deployment.md](docs/deployment.md) for environment-specific hardening notes.

## Testing
See [docs/testing.md](docs/testing.md) for the testing strategy, how to run tests locally, and coverage notes.

## Deployment
See [docs/deployment.md](docs/deployment.md) for deployment steps, environment configuration, and hosting recommendations.

## Performance Optimizations
- Tailwind CSS v4 with an integrated MSBuild build step keeps CSS output minimal (utility-first, purged classes).
- `<template>` elements are used for the lesson player to clone content client-side instead of re-rendering server-side.
- DOM lookups in interactive scripts (e.g., `lesson-progress.js`) use targeted `document.querySelector` calls to avoid unnecessary cross-subtree traversal.

## Known Issues
- Track open issues at: https://github.com/kh1aled/borhanix/issues
- The project is currently positioned as portfolio/freelance-ready rather than SaaS-viable (no multi-tenancy or payments yet).

## Future Improvements
- Payment integration (e.g., Stripe, and regional gateways such as Fawry/Paymob) for paid courses
- Multi-tenancy / white-label support
- Niche positioning for training centers in the MENA region
- Arabic/RTL support
- Expanded automated test coverage

## Contributing
Contributions are welcome. To contribute:
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes with clear messages
4. Open a Pull Request describing the change

Please keep Razor view changes consistent with existing ASP.NET tag helper patterns (`asp-for`, `asp-action`, `asp-route-*`) and the Tailwind design system defined in `tailwind.extension.json`.

## License
_Add license information (e.g., MIT, Apache 2.0) here. No license file is currently published in the repository — add a `LICENSE` file to formally declare terms of use._

## Authors
- **kh1aled** — [github.com/kh1aled](https://github.com/kh1aled)

## Acknowledgements
- [ASP.NET Core](https://dotnet.microsoft.com/apps/aspnet) and Entity Framework Core teams
- [Tailwind CSS](https://tailwindcss.com/)
- [Lucide Icons](https://lucide.dev/)
- [Toastify.js](https://apvarun.github.io/toastify-js/)
- [DeepSeek](https://www.deepseek.com/) / [OpenRouter](https://openrouter.ai/) for AI assistant capabilities

## Contact
For questions or support, open an issue on the repository: https://github.com/kh1aled/borhanix/issues