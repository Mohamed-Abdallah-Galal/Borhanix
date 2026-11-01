# Installation Guide

This guide walks through setting up Borhanix (internally `DepiLms`) for local development.

## Prerequisites
| Requirement | Version | Notes |
|---|---|---|
| .NET SDK | 8.0+ | Required to build/run the ASP.NET Core MVC app |
| Node.js & npm | 18+ | Required for the Tailwind CSS v4 build pipeline |
| SQL Server | LocalDB / Developer / Standard | Used via Entity Framework Core |
| Git | Any recent version | For cloning the repository |
| IDE | Visual Studio 2022, VS Code, or Rider | Optional but recommended |

Verify your tooling:
```bash
dotnet --version
node --version
npm --version
git --version
```

## 1. Clone the Repository
```bash
git clone https://github.com/kh1aled/borhanix.git
cd borhanix
```

## 2. Install Backend Dependencies
```bash
dotnet restore
```

## 3. Install Frontend (Tailwind) Dependencies
```bash
npm install
```

This installs the Tailwind CSS v4 CLI and any other JavaScript tooling declared in `package.json`.

## 4. Configure Environment Variables
Copy/update `appsettings.Development.json` with local values. At minimum you'll need:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=BorhanixDb;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
  "OpenRouter": {
    "ApiKey": "<your-openrouter-api-key>",
    "Model": "deepseek/deepseek-chat"
  }
}
```

**Recommended**: keep secrets out of `appsettings.json` using .NET User Secrets for local development:
```bash
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "<your-connection-string>"
dotnet user-secrets set "OpenRouter:ApiKey" "<your-api-key>"
```

## 5. Database Setup
Apply existing migrations to create the schema:
```bash
dotnet ef database update
```

If the EF Core CLI tool isn't installed globally:
```bash
dotnet tool install --global dotnet-ef
```

To create a new migration after changing a model:
```bash
dotnet ef migrations add <DescriptiveMigrationName>
dotnet ef database update
```

## 6. Build the Tailwind CSS
If Tailwind isn't wired into your MSBuild step automatically, run the watcher in a separate terminal during development:
```bash
npx tailwindcss -i ./wwwroot/css/input.css -o ./wwwroot/css/output.css --watch
```
(Adjust input/output paths to match the project's actual Tailwind configuration.)

## 7. Run the Application
```bash
dotnet run
```

Or, from your IDE, run/debug the `DepiLms` project. The app will be available at the URL configured in `Properties/launchSettings.json` (typically `https://localhost:5001`).

## 8. Verify the Setup
- Navigate to the home page and confirm it loads with Tailwind styles applied.
- Register a new account and confirm ASP.NET Identity is working.
- Log in as each role (Student/Instructor/Admin) to confirm role-based dashboards render correctly (roles/seed data may need to be created manually if no seeding script exists yet).

## Troubleshooting
| Issue | Likely Cause | Fix |
|---|---|---|
| `dotnet ef` command not found | EF Core tools not installed | `dotnet tool install --global dotnet-ef` |
| Styles not applying | Tailwind not built/watching | Run the Tailwind CLI build/watch command |
| Database connection errors | Wrong connection string or SQL Server not running | Verify `ConnectionStrings:DefaultConnection` and that your SQL Server instance is reachable |
| AI assistant not responding | Missing/invalid OpenRouter API key | Set `OpenRouter:ApiKey` via User Secrets or environment variables |

## Next Steps
- [Database Design](database.md)
- [API Documentation](api.md)
- [Deployment Guide](deployment.md)
