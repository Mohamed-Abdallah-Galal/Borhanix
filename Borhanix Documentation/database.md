# Database Design

Borhanix uses **SQL Server** as its database, accessed through **Entity Framework Core** with a code-first approach. Schema changes are tracked in the `Migrations/` folder and applied with `dotnet ef database update`.

> ⚠️ The tables and columns below are based on the application's known domain/features. Confirm and refine field names/types against the actual classes in `Models/` and the generated migrations in `Migrations/` before treating this as the authoritative schema.

## Entity Relationship Diagram (ERD)
See [diagrams/erd.png](diagrams/erd.png).

_To generate the ERD: use a tool such as EF Core Power Tools, dbdiagram.io, or `dotnet-ef dbcontext scaffold`/model visualization, then export as `erd.png` into `docs/diagrams/`._

## Core Entities

### Users (ASP.NET Identity)
| Column | Type | Notes |
|---|---|---|
| Id | string (GUID) | Primary key, Identity default |
| UserName / Email | string | Identity default |
| PasswordHash | string | Identity default |
| FullName | string | Custom profile field |
| ProfileImageUrl | string | Optional |
| Role | Student / Instructor / Admin | Managed via `AspNetRoles` / `AspNetUserRoles` |

### Courses
| Column | Type | Notes |
|---|---|---|
| Id | int/GUID | Primary key |
| Title | string | |
| Description | string | |
| InstructorId | FK → Users | Course owner |
| CategoryId | FK → Categories | Optional grouping |
| CoverImageUrl | string | |
| Price | decimal | For future payment integration |
| CreatedAt / UpdatedAt | datetime | |
| IsPublished | bool | Draft vs. live |

### Lessons / Course Content
| Column | Type | Notes |
|---|---|---|
| Id | int/GUID | Primary key |
| CourseId | FK → Courses | |
| Title | string | |
| ContentType | enum (Video, Text, Quiz, Assignment) | |
| ContentUrl / Body | string | |
| Order | int | Sequencing within the course |

### Enrollments
| Column | Type | Notes |
|---|---|---|
| Id | int/GUID | Primary key |
| StudentId | FK → Users | |
| CourseId | FK → Courses | |
| Status | enum (Pending, Approved, Rejected) | Admin/instructor approval workflow |
| EnrolledAt | datetime | |
| ProgressPercentage | decimal/int | |

### Quizzes & Assignments
| Column | Type | Notes |
|---|---|---|
| Id | int/GUID | Primary key |
| CourseId / LessonId | FK | Association to course content |
| Title | string | |
| Type | enum (Quiz, Assignment) | |
| DueDate | datetime | Assignments |
| MaxScore | int | |

### Submissions / Results
| Column | Type | Notes |
|---|---|---|
| Id | int/GUID | Primary key |
| QuizAssignmentId | FK | |
| StudentId | FK → Users | |
| Score | int | |
| SubmittedAt | datetime | |

### Attendance Records
| Column | Type | Notes |
|---|---|---|
| Id | int/GUID | Primary key |
| StudentId | FK → Users | |
| SessionId / CourseId | FK | Which session/course the attendance applies to |
| ScannedAt | datetime | Captured at QR scan time |
| Method | enum (QR) | |

### Certificates
| Column | Type | Notes |
|---|---|---|
| Id | int/GUID | Primary key |
| StudentId | FK → Users | |
| CourseId | FK → Courses | |
| IssuedAt | datetime | |
| CertificateUrl | string | Generated certificate file/link |

## Relationships Summary
- A **User** (Instructor) has many **Courses**.
- A **Course** has many **Lessons/Content items**.
- A **Course** has many **Enrollments**, each linking to one **Student**.
- A **Lesson** may have an associated **Quiz/Assignment**.
- A **Student** has many **Submissions** and **Attendance Records**.
- A **Student** receives a **Certificate** per completed **Course**.

## Migrations
Migrations live under `Migrations/` and are the source of truth for the actual applied schema. Common commands:

```bash
# Add a new migration after changing a model
dotnet ef migrations add <MigrationName>

# Apply pending migrations to the database
dotnet ef database update

# Remove the last (unapplied) migration
dotnet ef migrations remove

# Generate a SQL script for a migration (e.g., for production deployment)
dotnet ef migrations script
```

## Indexing & Performance Notes
- Foreign key columns (`InstructorId`, `CourseId`, `StudentId`) should be indexed to keep dashboard and enrollment queries fast.
- Consider a composite index on `(CourseId, StudentId)` for `Enrollments` to speed up "is this student enrolled" checks.

## Related Documents
- [Architecture](architecture.md)
- [API Documentation](api.md)
