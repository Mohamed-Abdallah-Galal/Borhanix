# Testing

## Current State
As of this documentation, Borhanix does not yet include a dedicated automated test project in the repository. This document defines the recommended testing strategy to adopt as the project matures toward a SaaS-viable product.

## Recommended Testing Strategy

### 1. Unit Tests
Target the **Services** layer (business logic) and any standalone helper/utility classes, since Controllers should stay thin.
- Framework: [xUnit](https://xunit.net/) (standard for .NET)
- Mocking: [Moq](https://github.com/moq/moq4) or [NSubstitute](https://nsubstitute.github.io/) for dependencies like the DbContext or the OpenRouter client
- Suggested project: `Borhanix.Tests` (or `DepiLms.Tests`), referencing the main project

Example structure:
```
DepiLms.Tests/
├── Services/
│   ├── EnrollmentServiceTests.cs
│   ├── AttendanceServiceTests.cs
│   └── CertificateServiceTests.cs
├── Controllers/
│   └── CourseControllerTests.cs
└── DepiLms.Tests.csproj
```

Example test:
```csharp
public class EnrollmentServiceTests
{
    [Fact]
    public async Task Enroll_WhenAlreadyEnrolled_ReturnsFailure()
    {
        // Arrange
        var service = new EnrollmentService(mockDbContext);

        // Act
        var result = await service.EnrollAsync(studentId, courseId);

        // Assert
        Assert.False(result.Success);
    }
}
```

### 2. Integration Tests
Use [`Microsoft.AspNetCore.Mvc.Testing`](https://learn.microsoft.com/aspnet/core/test/integration-tests) with `WebApplicationFactory<Program>` to spin up the app in-memory and test controller behavior end-to-end, backed by an EF Core **in-memory** or **SQLite in-memory** database for isolation.

Priority areas for integration coverage:
- Authentication/authorization flows (Register, Login, role-restricted routes)
- Enrollment workflow (request → approval → access to course content)
- Progress tracking on the Learn page
- Attendance recording via QR flow
- Certificate issuance on course completion

### 3. Manual/Exploratory Testing Checklist
Until automated coverage is built out, use this checklist before each release:

**Authentication**
- [ ] Register as a new user (defaults to Student)
- [ ] Login/logout works
- [ ] Access-denied page shows for wrong-role access attempts

**Student Flow**
- [ ] Browse course catalog and view course details
- [ ] Enroll in a course
- [ ] Progress through lessons on the Learn page
- [ ] Take a quiz / submit an assignment
- [ ] Receive a certificate on completion
- [ ] Scan/display QR attendance ID card

**Instructor Flow**
- [ ] Create a new course
- [ ] Add/reorder lesson content in Manage Content
- [ ] Duplicate an existing course
- [ ] Grade a student submission
- [ ] Scan student attendance QR codes

**Admin Flow**
- [ ] View Admin Dashboard
- [ ] Approve/reject a pending enrollment
- [ ] Manage user accounts/roles

**Cross-cutting**
- [ ] Dark mode toggles correctly across pages
- [ ] Toast notifications appear for key actions (success/error)
- [ ] AI assistant responds to a test question

## Running Tests (once a test project exists)
```bash
dotnet test
```

## Continuous Integration
Recommended: add a GitHub Actions workflow that runs `dotnet build` and `dotnet test` on every push/PR to `main`, e.g. `.github/workflows/ci.yml`:
```yaml
name: CI
on: [push, pull_request]
jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '8.0.x'
      - run: dotnet restore
      - run: dotnet build --no-restore
      - run: dotnet test --no-build
```

## Related Documents
- [Architecture](architecture.md)
- [Deployment](deployment.md)
