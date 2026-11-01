# Deployment Guide

This guide covers deploying Borhanix (`DepiLms`) to a production or staging environment.

## Deployment Options
Borhanix, being a standard ASP.NET Core MVC app with a SQL Server backend, can be deployed to:
- **Azure App Service** (recommended for simplicity, with Azure SQL Database)
- **IIS** on a Windows Server (self-hosted)
- **Linux VM / Docker container** using Kestrel behind a reverse proxy (e.g., Nginx)
- Any container platform (with a Dockerfile added to the project)

## 1. Pre-Deployment Checklist
- [ ] All EF Core migrations are committed and tested
- [ ] `appsettings.Production.json` (or environment variables) configured with production connection string and API keys
- [ ] OpenRouter API key set as a secret (not in source control)
- [ ] Tailwind CSS built for production (`npm run build` / production Tailwind config, minified output)
- [ ] HTTPS certificate configured
- [ ] Database backups configured

## 2. Build for Production
```bash
dotnet publish -c Release -o ./publish
```

Build the production Tailwind CSS bundle before or as part of publish:
```bash
npx tailwindcss -i ./wwwroot/css/input.css -o ./wwwroot/css/output.css --minify
```

## 3. Database Migration in Production
Apply migrations against the production database (run this in a controlled deployment step, not automatically on every app start):
```bash
dotnet ef database update --connection "<production-connection-string>"
```
Alternatively, generate a SQL script and run it through your normal DB change-management process:
```bash
dotnet ef migrations script --idempotent -o migrate.sql
```

## 4. Environment Configuration
Set the following via environment variables or your hosting platform's secret manager (do **not** hardcode in `appsettings.json`):

| Setting | Description |
|---|---|
| `ConnectionStrings__DefaultConnection` | Production SQL Server connection string |
| `OpenRouter__ApiKey` | API key for the DeepSeek AI assistant integration |
| `ASPNETCORE_ENVIRONMENT` | Set to `Production` |

## 5. Deploying to Azure App Service (example)
```bash
az webapp up \
  --name <your-app-name> \
  --resource-group <your-resource-group> \
  --runtime "DOTNETCORE:8.0"
```
Then configure application settings (connection strings, API keys) in the Azure Portal under **Configuration**, and point `ConnectionStrings:DefaultConnection` to an Azure SQL Database.

## 6. Deploying to IIS (example)
1. Install the [ASP.NET Core Hosting Bundle](https://dotnet.microsoft.com/download/dotnet/8.0) on the Windows Server.
2. Publish the app (`dotnet publish -c Release`) and copy the output to the IIS site's physical path.
3. Create an IIS Application Pool set to **No Managed Code** (ASP.NET Core is self-hosted via Kestrel behind IIS as a reverse proxy).
4. Bind HTTPS with a valid certificate.
5. Ensure the app pool identity has read access to the deployment folder and connectivity to SQL Server.

## 7. Reverse Proxy (Linux/Nginx example)
Run the app with Kestrel and place Nginx in front:
```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass         http://127.0.0.1:5000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection keep-alive;
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header   X-Forwarded-Proto $scheme;
    }
}
```
Run the app as a `systemd` service so it restarts automatically:
```ini
[Unit]
Description=Borhanix LMS

[Service]
WorkingDirectory=/var/www/borhanix
ExecStart=/usr/bin/dotnet /var/www/borhanix/DepiLms.dll
Restart=always
RestartSec=10
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production

[Install]
WantedBy=multi-user.target
```

## 8. Post-Deployment Verification
- [ ] Home page loads over HTTPS
- [ ] Registration/login works end-to-end
- [ ] Each role's dashboard renders correctly
- [ ] Database reads/writes succeed (e.g., enroll in a course)
- [ ] AI assistant responds (verifies OpenRouter connectivity/API key)
- [ ] Static assets (Tailwind CSS, images, Lucide icons) load correctly

## Rollback Plan
- Keep the previous published build available to redeploy quickly if a release fails.
- For database changes, prefer additive/backward-compatible migrations so a code rollback doesn't require an immediate schema rollback.

## Related Documents
- [Installation Guide](installation.md)
- [Database Design](database.md)
