# API Documentation

Borhanix is primarily a server-rendered ASP.NET Core MVC application, so most "endpoints" are MVC controller actions returning HTML Views rather than a JSON REST API. This document covers both:
1. **MVC routes** (Controller/Action) used for page navigation and form submissions.
2. **AJAX/JSON endpoints**, where present (e.g., for the AI assistant chat, progress updates, or attendance scanning), which do return JSON.

> ⚠️ Route names and exact payloads below are based on the application's known feature set (see `Controllers/`). Verify exact route names, HTTP verbs, and DTOs against the actual controller actions before publishing this as a public API reference.

## Authentication
Authentication is handled by **ASP.NET Identity**, using cookie-based authentication for the MVC app (not token-based, unless a separate API surface is added later).

| Action | Route (typical) | Method | Description |
|---|---|---|---|
| Register | `/Account/Register` | GET/POST | Create a new account (default role: Student) |
| Login | `/Account/Login` | GET/POST | Authenticate and set the auth cookie |
| Logout | `/Account/Logout` | POST | Clear the auth cookie |
| Access Denied | `/Account/AccessDenied` | GET | Shown when a user lacks the required role |

Role-protected actions are annotated with `[Authorize(Roles = "Student,Instructor,Admin")]` (adjusted per action) at the controller/action level.

## Endpoints

### Course Endpoints
| Action | Route (typical) | Method | Role(s) | Description |
|---|---|---|---|---|
| Course Catalog | `/Course/Index` | GET | Public/Student | List/browse published courses |
| Course Details | `/Course/Details/{id}` | GET | Public/Student | View a single course's details |
| Enroll | `/Course/Enroll/{id}` | POST | Student | Request enrollment in a course |
| Create Course | `/Instructor/Course/Create` | GET/POST | Instructor | Create a new course |
| Edit Course | `/Instructor/Course/Edit/{id}` | GET/POST | Instructor | Edit course details |
| Duplicate Course | `/Instructor/Course/Duplicate/{id}` | POST | Instructor | Deep-copy a course and its content |
| Manage Content | `/Instructor/ManageContent/{courseId}` | GET/POST | Instructor | Add/reorder/edit lessons within a course |

### Learning & Progress Endpoints
| Action | Route (typical) | Method | Role(s) | Description |
|---|---|---|---|---|
| Learn Page | `/Learn/{courseId}` | GET | Student | Lesson player for an enrolled course |
| Update Progress | `/Learn/UpdateProgress` | POST (AJAX/JSON) | Student | Persist lesson completion/progress |
| Certificate | `/Certificate/{courseId}` | GET | Student | View/download the completion certificate |

### Quiz & Assignment Endpoints
| Action | Route (typical) | Method | Role(s) | Description |
|---|---|---|---|---|
| Take Quiz | `/Quiz/{id}` | GET/POST | Student | View and submit a quiz |
| Submit Assignment | `/Assignment/{id}/Submit` | POST | Student | Submit an assignment |
| Grade Submission | `/Instructor/Grading/{submissionId}` | GET/POST | Instructor | Review and grade a submission |

### Attendance Endpoints
| Action | Route (typical) | Method | Role(s) | Description |
|---|---|---|---|---|
| Scanner View | `/Attendance/Scan` | GET | Instructor | QR scanner UI for a session |
| Record Attendance | `/Attendance/Record` | POST (AJAX/JSON) | Instructor | Record a scanned student's attendance |
| Student ID Card | `/Attendance/MyCard` | GET | Student | View the student's own QR ID card |

### AI Assistant Endpoint
| Action | Route (typical) | Method | Role(s) | Description |
|---|---|---|---|---|
| Chat | `/AiAssistant/Chat` | POST (AJAX/JSON) | Student (Authenticated) | Sends a message to the DeepSeek assistant via OpenRouter and returns the reply |

### Admin Endpoints
| Action | Route (typical) | Method | Role(s) | Description |
|---|---|---|---|---|
| Admin Dashboard | `/Admin/Index` | GET | Admin | Platform overview |
| User Control | `/Admin/Users` | GET/POST | Admin | Manage users and roles |
| Enrollment Approvals | `/Admin/Enrollments` | GET/POST | Admin | Approve/reject pending enrollments |

## Request & Response Examples

### Example: Update Lesson Progress (AJAX/JSON)
**Request**
```http
POST /Learn/UpdateProgress
Content-Type: application/json

{
  "lessonId": 42,
  "courseId": 7,
  "completed": true
}
```

**Response**
```json
{
  "success": true,
  "progressPercentage": 65
}
```

### Example: AI Assistant Chat
**Request**
```http
POST /AiAssistant/Chat
Content-Type: application/json

{
  "message": "Can you explain closures in JavaScript?"
}
```

**Response**
```json
{
  "reply": "A closure is a function that retains access to variables from its enclosing scope even after that scope has finished executing..."
}
```

### Example: Record Attendance (AJAX/JSON)
**Request**
```http
POST /Attendance/Record
Content-Type: application/json

{
  "sessionId": 12,
  "studentQrToken": "eyJhbGciOi..."
}
```

**Response**
```json
{
  "success": true,
  "studentName": "Jane Doe",
  "scannedAt": "2026-08-06T10:15:00Z"
}
```

## Error Handling
Standard MVC error handling applies:
- Invalid form submissions return the same View with validation errors (via `ModelState`).
- Unauthenticated requests to protected routes redirect to `/Account/Login`.
- Unauthorized (wrong role) requests redirect to `/Account/AccessDenied`.
- AJAX/JSON endpoints should return a consistent error shape, e.g.:
```json
{
  "success": false,
  "error": "You are not enrolled in this course."
}
```

## Related Documents
- [Architecture](architecture.md)
- [Database Design](database.md)
