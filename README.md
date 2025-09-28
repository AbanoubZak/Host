# Comprehensive Hosting Guide: Integral Solution on Windows Server 2019 Standard

### Overview

Your Integral solution uses .NET 9 with Clean Architecture, featuring:

- Backend API with FastEndpoints and JWT authentication
- Blazor Server frontend with role-based permissions
- PostgreSQL database with EF Core migrations
- Advanced authentication system with session management

**Prerequisites Status:**

- ❌ IIS (Internet Information Services) - Not installed
- ❌ PostgreSQL Database Server - Not installed  
- ❌ .NET 9 Runtime and Hosting Bundle - Not installed
- ❌ Visual C++ Redistributables - May not be installed

This guide will walk you through installing everything from scratch on a fresh Windows Server 2019 Standard installation.

## Installation Order Summary

**Follow this exact order for best results:**

1. **Install IIS** using Server Manager (Step 1.1)
2. **Install Visual C++ Redistributables** (Step 1.2)  
3. **Install .NET 9 Hosting Bundle** (Step 1.3)
4. **Install PostgreSQL** (Step 2.1)
5. **Configure PostgreSQL** (Steps 2.2-2.4)
6. **Configure IIS Sites** (Step 3)
7. **Deploy Applications** (Step 4)
8. **Security Hardening** (Step 5)

**Estimated Total Installation Time:** 45-60 minutes

---

## Step 1: Windows Server 2019 Standard Prerequisites

### 1.1 Install IIS (Internet Information Services)

**Open Server Manager:**

1. Click **Start** → **Server Manager**
2. Click **Add roles and features**
3. Click **Next** until you reach **Server Roles**
4. Check **Web Server (IIS)**
5. In the popup, click **Add Features**
6. Click **Next** until you reach **Role Services**

**Select Required IIS Features:**

Under **Web Server (IIS)** → **Web Server** → **Application Development**, select:

- ✅ **ASP.NET 4.8** (and all dependencies)
- ✅ **.NET Extensibility 4.8**
- ✅ **ISAPI Extensions**
- ✅ **ISAPI Filters**
- ✅ **WebSocket Protocol**

Under **Web Server (IIS)** → **Web Server** → **Common HTTP Features**, ensure selected:

- ✅ **Default Document**
- ✅ **Directory Browsing**
- ✅ **HTTP Errors**
- ✅ **HTTP Redirection**
- ✅ **Static Content**

Under **Web Server (IIS)** → **Web Server** → **Security**, ensure selected:

- ✅ **Request Filtering**

Under **Web Server (IIS)** → **Management Tools**, ensure selected:

- ✅ **IIS Management Console**

1. Click **Next** → **Install**
2. **Restart the server** after installation completes

**Verify IIS Installation:**

```powershell
# Open PowerShell as Administrator and run:
Get-WindowsFeature -Name IIS-*
# Should show multiple IIS features as "Installed"

# Test IIS is working
Start-Process "http://localhost"
# Should open default IIS page in browser
```

### 1.2 Install Visual C++ Redistributables (Required for .NET and PostgreSQL)

**Download and Install:**

1. **Microsoft Visual C++ 2015-2022 Redistributable (x64)**
   - Download from: <https://aka.ms/vs/17/release/vc_redist.x64.exe>
   - Run the installer and follow the setup wizard

2. **Microsoft Visual C++ 2015-2022 Redistributable (x86)**
   - Download from: <https://aka.ms/vs/17/release/vc_redist.x86.exe>  
   - Run the installer and follow the setup wizard

3. **Restart the server** after both installations complete

### 1.3 Install .NET 9 Runtime and Hosting Bundle

**Download .NET 9 Hosting Bundle:**

1. Open a web browser and navigate to: <https://dotnet.microsoft.com/download/dotnet/9.0>
2. Under **ASP.NET Core Runtime 9.0.x**, click **Download Hosting Bundle** (Windows x64)
3. Save the file (typically named `dotnet-hosting-9.x.x-win.exe`) to the server

**Install the Hosting Bundle:**

1. Run the downloaded installer as Administrator
2. Follow the installation wizard (accept defaults)
3. **Important:** Restart IIS after installation:

```powershell
# Open PowerShell as Administrator
net stop was /y
net start w3svc
# Or restart the entire server for best results
```

**Verify .NET 9 Installation:**

```powershell
# Check installed runtimes
dotnet --info
dotnet --list-runtimes

# You should see something like:
# Microsoft.AspNetCore.App 9.x.x
# Microsoft.NETCore.App 9.x.x
```

### 1.4 Enable Additional Windows Features (PowerShell Method)

**Run PowerShell as Administrator and execute:**

**Download and Install .NET 9 Runtime:**

1. Download the latest .NET 9 Hosting Bundle from: <https://dotnet.microsoft.com/download/dotnet/9.0>
2. Run the installer: `dotnet-hosting-9.x.x-win.exe`
3. Restart IIS after installation:

   ```powershell
   net stop was /y
   net start w3svc
   ```

**Verify Installation:**

```powershell
dotnet --info
dotnet --list-runtimes
```

### 1.2 Enable Windows Features

**Using PowerShell (Run as Administrator):**

```powershell
# Enable IIS and ASP.NET Core features
Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebServerRole
Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebServer
Enable-WindowsOptionalFeature -Online -FeatureName IIS-CommonHttpFeatures
Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpErrors
Enable-WindowsOptionalFeature -Online -FeatureName IIS-HttpLogging
Enable-WindowsOptionalFeature -Online -FeatureName IIS-Security
Enable-WindowsOptionalFeature -Online -FeatureName IIS-RequestFiltering
Enable-WindowsOptionalFeature -Online -FeatureName IIS-StaticContent
Enable-WindowsOptionalFeature -Online -FeatureName IIS-DefaultDocument
Enable-WindowsOptionalFeature -Online -FeatureName IIS-DirectoryBrowsing
Enable-WindowsOptionalFeature -Online -FeatureName IIS-WebSockets
Enable-WindowsOptionalFeature -Online -FeatureName IIS-ApplicationDevelopment
Enable-WindowsOptionalFeature -Online -FeatureName IIS-ASPNET45

# Restart required
Restart-Computer
```

### 1.3 Install Visual C++ Redistributables

Download and install both x64 and x86 versions:

- Microsoft Visual C++ 2015-2022 Redistributable (x64)
- Microsoft Visual C++ 2015-2022 Redistributable (x86)

---

## Step 2: PostgreSQL Installation and Configuration

### 2.1 Download and Install PostgreSQL

**Download PostgreSQL:**

1. Open a web browser and navigate to: <https://www.postgresql.org/download/windows/>
2. Click **Download the installer** → **Download** for the latest PostgreSQL 16.x version
3. Save the installer (typically named `postgresql-16.x-x-windows-x64.exe`) to the server

**Run the PostgreSQL Installer:**

1. **Right-click** the installer and select **Run as administrator**
2. Follow the installation wizard with these recommended settings:
   - **Installation Directory:** `C:\Program Files\PostgreSQL\16` (default)
   - **Data Directory:** `C:\Program Files\PostgreSQL\16\data` (default)
   - **Password:** Enter a **strong password** for the postgres superuser (write this down!)
   - **Port:** `5432` (default)
   - **Locale:** `English, United States` (default)
   - **Stack Builder:** Uncheck (not needed for basic installation)
3. Click **Next** and **Install**
4. Wait for installation to complete (may take several minutes)

**Important Notes:**

- Remember the postgres superuser password - you'll need it later!
- The installer will automatically create a Windows service that starts with the system

### 2.2 Configure PostgreSQL Service

**Check PostgreSQL Service:**

```powershell
# Verify service is running
Get-Service -Name postgresql*
```

**Configure Service for Auto-Start:**

```powershell
# Set service to start automatically
Set-Service -Name postgresql-x64-16 -StartupType Automatic
Start-Service -Name postgresql-x64-16
```

### 2.3 Create Database and User

**Connect to PostgreSQL:**

```powershell
# Open PostgreSQL command line
cd "C:\Program Files\PostgreSQL\16\bin"
.\psql.exe -U postgres
```

**Create Database and User:**

```sql
-- Create database for Integral
CREATE DATABASE integral_db;

-- Create application user
CREATE USER integral_user WITH PASSWORD 'your_secure_password_here';

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE integral_db TO integral_user;

-- Connect to the new database
\c integral_db;

-- Grant schema privileges
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO integral_user;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO integral_user;
GRANT USAGE ON SCHEMA public TO integral_user;

-- Set default privileges for future objects
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL PRIVILEGES ON TABLES TO integral_user;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL PRIVILEGES ON SEQUENCES TO integral_user;

-- Exit
\q
```

### 2.4 Configure PostgreSQL Security

**Edit pg_hba.conf:**

```powershell
# Open configuration file
notepad "C:\Program Files\PostgreSQL\16\data\pg_hba.conf"
```

**Add these lines for local connections only:**

```conf
# TYPE  DATABASE        USER            ADDRESS                 METHOD
# Local connections only - no remote access
local   integral_db     integral_user                          md5
host    integral_db     integral_user   127.0.0.1/32           md5
host    integral_db     integral_user   ::1/128                md5

# Reject all other connections for security
host    all             all             0.0.0.0/0               reject
```

**Restart PostgreSQL:**

```powershell
Restart-Service -Name postgresql-x64-16
```

---

## Step 3: IIS Configuration

### 3.1 Create Application Pools

**Open IIS Manager** (`inetmgr.exe`)

**Create Application Pool for Backend API:**

1. Right-click `Application Pools` → `Add Application Pool`
2. **Name:** `IntegralBackendAPI`
3. **.NET CLR Version:** `No Managed Code`
4. **Managed Pipeline Mode:** `Integrated`
5. **Start immediately:** ✓

**Advanced Settings for Backend API Pool:**

- **Process Model → Identity:** `ApplicationPoolIdentity`
- **Process Model → Idle Timeout:** `0` (Never timeout)
- **Recycling → Regular Time Interval:** `0` (Disable recycling)

**Create Application Pool for Frontend UI:**

1. Right-click `Application Pools` → `Add Application Pool`
2. **Name:** `IntegralFrontendUI`
3. **.NET CLR Version:** `No Managed Code`
4. **Managed Pipeline Mode:** `Integrated`
5. **Start immediately:** ✓

**Advanced Settings for Frontend UI Pool:**

- Same settings as Backend API pool

### 3.2 Create Websites and Applications

**Create Backend API Website:**

1. Right-click `Sites` → `Add Website`
2. **Site name:** `Integral-Backend-API`
3. **Application pool:** `IntegralBackendAPI`
4. **Physical path:** `C:\inetpub\IntegralBackend`
5. **Port:** `8080` (HTTP)
6. **Host name:** `api.integral.local` (we'll configure this custom domain)

**Create Frontend UI Website:**

1. Right-click `Sites` → `Add Website`
2. **Site name:** `Integral-Frontend-UI`
3. **Application pool:** `IntegralFrontendUI`
4. **Physical path:** `C:\inetpub\IntegralFrontend`
5. **Port:** `8081` (HTTP)
6. **Host name:** `app.integral.local` (we'll configure this custom domain)

### 3.3 Configure ASP.NET Core Module

**Install ASP.NET Core Module** (should be installed with Hosting Bundle)

**Configure web.config for Backend API** (will be generated during publish, but here's the template):

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet"
                  arguments=".\Integral.Backend.API.dll"
                  stdoutLogEnabled="false"
                  stdoutLogFile=".\logs\stdout"
                  hostingModel="InProcess" />
    </system.webServer>
  </location>
</configuration>
```

---

## Step 4: Application Deployment

### 4.1 Prepare for Production Deployment

**On your development machine**, create production configuration files:

**Backend API - appsettings.Production.json:**

```json
{
  "ConnectionStrings": {
    "PostgresConnection": "Host=localhost;Port=5432;Database=integral_db;Username=integral_user;Password=your_secure_password_here;SSL Mode=Disable;"
  },
  "Serilog": {
    "Using": ["Serilog.Sinks.Console", "Serilog.Sinks.File"],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "Microsoft.AspNetCore": "Warning"
      }
    },
    "WriteTo": [
      {
        "Name": "Console"
      },
      {
        "Name": "File",
        "Args": {
          "path": "C:\\Logs\\IntegralBackend\\log-.txt",
          "rollingInterval": "Day",
          "retainedFileCountLimit": 30
        }
      }
    ],
    "Enrich": ["FromLogContext"],
    "Properties": {
      "Application": "Integral.Backend.API"
    }
  },
  "AllowedHosts": "api.integral.local;localhost",
  "JwtSettings": {
    "Key": "your_production_secret_key_at_least_32_characters_long_and_secure",
    "Issuer": "Integral.Backend.API",
    "Audience": "Integral.Clients",
    "DurationInMinutes": "480"
  },
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:5000"
      },
      "Https": {
        "Url": "https://localhost:5443"
      }
    }
  }
}
```

**Frontend UI - appsettings.Production.json:**

```json
{
  "Serilog": {
    "Using": ["Serilog.Sinks.Console", "Serilog.Sinks.File"],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "Microsoft.AspNetCore": "Warning"
      }
    },
    "WriteTo": [
      {
        "Name": "Console"
      },
      {
        "Name": "File",
        "Args": {
          "path": "C:\\Logs\\IntegralFrontend\\log-.txt",
          "rollingInterval": "Day",
          "retainedFileCountLimit": 30
        }
      }
    ]
  },
  "AllowedHosts": "app.integral.local;localhost",
  "ApiSettings": {
    "FrontendBaseUrl": "https://app.integral.local:8444",
    "BackendBaseUrl": "https://api.integral.local:8443"
  },
  "JwtSettings": {
    "Key": "your_production_secret_key_at_least_32_characters_long_and_secure",
    "Issuer": "Integral.Backend.API",
    "Audience": "Integral.Clients"
  },
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:5001"
      },
      "Https": {
        "Url": "https://localhost:5444"
      }
    }
  }
}
```

### 4.2 Publish Applications

**Publish Backend API:**

```powershell
# Navigate to your project directory
cd "D:\Private Projects\Integral"

# Publish Backend API
dotnet publish "Integral.Backend.API\Integral.Backend.API.csproj" `
  --configuration Release `
  --runtime win-x64 `
  --self-contained false `
  --output "C:\Publish\IntegralBackend"
```

**Publish Frontend UI:**

```powershell
# Publish Frontend UI
dotnet publish "Integral.Frontend.UI\Integral.Frontend.UI.csproj" `
  --configuration Release `
  --runtime win-x64 `
  --self-contained false `
  --output "C:\Publish\IntegralFrontend"
```

### 4.3 Deploy to Windows Server

**Create Directory Structure:**

```powershell
# Create application directories
New-Item -Path "C:\inetpub\IntegralBackend" -ItemType Directory -Force
New-Item -Path "C:\inetpub\IntegralFrontend" -ItemType Directory -Force

# Create log directories
New-Item -Path "C:\Logs\IntegralBackend" -ItemType Directory -Force
New-Item -Path "C:\Logs\IntegralFrontend" -ItemType Directory -Force

# Set permissions for IIS
icacls "C:\inetpub\IntegralBackend" /grant "IIS_IUSRS:(OI)(CI)F" /t
icacls "C:\inetpub\IntegralFrontend" /grant "IIS_IUSRS:(OI)(CI)F" /t
icacls "C:\Logs\IntegralBackend" /grant "IIS_IUSRS:(OI)(CI)F" /t
icacls "C:\Logs\IntegralFrontend" /grant "IIS_IUSRS:(OI)(CI)F" /t
```

**Copy Published Files:**

1. Copy all files from `C:\Publish\IntegralBackend\*` to `C:\inetpub\IntegralBackend\`
2. Copy all files from `C:\Publish\IntegralFrontend\*` to `C:\inetpub\IntegralFrontend\`

### 4.4 Database Migration

**Run EF Core Migrations:**

```powershell
# Navigate to backend directory
cd "C:\inetpub\IntegralBackend"

# Set environment variable
$env:ASPNETCORE_ENVIRONMENT = "Production"

# Update database (this will create tables and seed data)
dotnet Integral.Backend.API.dll --migrate-database
```

**Note:** Your application has auto-migration in Program.cs, but you may need to run this manually

### 4.5 Configure Environment Variables

**For Backend API Application Pool:**

1. Open IIS Manager → Application Pools → IntegralBackendAPI
2. Advanced Settings → Process Model → Environment Variables
3. Add:
   - `ASPNETCORE_ENVIRONMENT` = `Production`
   - `DOTNET_ENVIRONMENT` = `Production`

**For Frontend UI Application Pool:**

1. Open IIS Manager → Application Pools → IntegralFrontendUI
2. Advanced Settings → Process Model → Environment Variables
3. Add:
   - `ASPNETCORE_ENVIRONMENT` = `Production`
   - `DOTNET_ENVIRONMENT` = `Production`

---

## Step 5: Security and Maintenance

Completed (6/6) *Create Security and Maintenance Guide*

### 5.1 Custom Local Domains and SSL Configuration

#### 5.1.1 Configure Custom Local Domains

**Update Windows Hosts File:**

1. Open **Notepad as Administrator**
2. Open file: `C:\Windows\System32\drivers\etc\hosts`
3. Add these lines at the end:

```conf
# Integral Application Local Domains
127.0.0.1    api.integral.local
127.0.0.1    app.integral.local
```

4. Save the file

**For Network Access from Other Computers:**

On client machines, add to their hosts file:

```conf
# Replace 192.168.1.100 with your server's IP address
192.168.1.100    api.integral.local
192.168.1.100    app.integral.local
```

#### 5.1.2 Generate Free Local SSL Certificates

**Create Self-Signed Certificates for Local Network:**

```powershell
# Run PowerShell as Administrator

# Create certificate for API
$apiCert = New-SelfSignedCertificate -Subject "CN=api.integral.local" `
    -DnsName "api.integral.local","localhost" `
    -CertStoreLocation "Cert:\LocalMachine\My" `
    -NotAfter (Get-Date).AddYears(5) `
    -KeyUsage DigitalSignature,KeyEncipherment `
    -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.1")

# Create certificate for Frontend
$frontendCert = New-SelfSignedCertificate -Subject "CN=app.integral.local" `
    -DnsName "app.integral.local","localhost" `
    -CertStoreLocation "Cert:\LocalMachine\My" `
    -NotAfter (Get-Date).AddYears(5) `
    -KeyUsage DigitalSignature,KeyEncipherment `
    -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.1")

# Export certificates to Trusted Root (makes them trusted)
$apiCertPath = "Cert:\LocalMachine\Root\" + $apiCert.Thumbprint
$frontendCertPath = "Cert:\LocalMachine\Root\" + $frontendCert.Thumbprint

Copy-Item -Path ("Cert:\LocalMachine\My\" + $apiCert.Thumbprint) -Destination "Cert:\LocalMachine\Root\"
Copy-Item -Path ("Cert:\LocalMachine\My\" + $frontendCert.Thumbprint) -Destination "Cert:\LocalMachine\Root\"

Write-Host "API Certificate Thumbprint: " $apiCert.Thumbprint
Write-Host "Frontend Certificate Thumbprint: " $frontendCert.Thumbprint
```

#### 5.1.3 Configure HTTPS Bindings in IIS

**Add HTTPS Binding for Backend API:**

1. Open IIS Manager
2. Select `Integral-Backend-API` site
3. Click **Bindings** in Actions panel
4. Click **Add**
5. **Type:** `https`
6. **IP Address:** `All Unassigned`
7. **Port:** `8443`
8. **Host name:** `api.integral.local`
9. **SSL certificate:** Select the certificate for `api.integral.local`
10. **Require Server Name Indication:** ✓
11. Click **OK**

**Add HTTPS Binding for Frontend UI:**

1. Select `Integral-Frontend-UI` site
2. Click **Bindings** in Actions panel
3. Click **Add**
4. **Type:** `https`
5. **IP Address:** `All Unassigned`
6. **Port:** `8444`
7. **Host name:** `app.integral.local`
8. **SSL certificate:** Select the certificate for `app.integral.local`
9. **Require Server Name Indication:** ✓
10. Click **OK**

### 5.2 Windows Firewall Configuration

```powershell
# Allow HTTP traffic for local network access
New-NetFirewallRule -DisplayName "Allow HTTP API" -Direction Inbound -Protocol TCP -LocalPort 8080
New-NetFirewallRule -DisplayName "Allow HTTP Frontend" -Direction Inbound -Protocol TCP -LocalPort 8081

# Allow HTTPS traffic for local network access
New-NetFirewallRule -DisplayName "Allow HTTPS API" -Direction Inbound -Protocol TCP -LocalPort 8443
New-NetFirewallRule -DisplayName "Allow HTTPS Frontend" -Direction Inbound -Protocol TCP -LocalPort 8444

# Note: PostgreSQL runs locally only - no external firewall rules needed
```

### 5.3 Security Hardening

**Application Pool Security:**

1. Run each app pool under dedicated service account
2. Grant minimum required permissions
3. Enable periodic recycling for security

**PostgreSQL Security:**

- Change default postgres user password
- Disable unnecessary PostgreSQL extensions
- Configure connection limits
- Enable SSL connections in production

**File System Permissions:**

```powershell
# Restrict access to application files
icacls "C:\inetpub\IntegralBackend\appsettings.Production.json" /grant "IIS_IUSRS:R" /inheritance:r
icacls "C:\inetpub\IntegralFrontend\appsettings.Production.json" /grant "IIS_IUSRS:R" /inheritance:r
```

### 5.4 Monitoring and Maintenance

**Create PowerShell Monitoring Script:**

```powershell
# Save as C:\Scripts\MonitorIntegral.ps1
# Monitor application health using custom domains
$backendUrl = "https://api.integral.local:8443/health"
$frontendUrl = "https://app.integral.local:8444"

try {
    # Skip certificate validation for self-signed certificates
    $backendResponse = Invoke-WebRequest -Uri $backendUrl -UseBasicParsing -SkipCertificateCheck
    $frontendResponse = Invoke-WebRequest -Uri $frontendUrl -UseBasicParsing -SkipCertificateCheck
    
    Write-Host "Backend Status: $($backendResponse.StatusCode)"
    Write-Host "Frontend Status: $($frontendResponse.StatusCode)"
    
    # Log to event log
    Write-EventLog -LogName Application -Source "Integral Monitor" -EventId 1000 -EntryType Information -Message "Applications are healthy"
} catch {
    Write-EventLog -LogName Application -Source "Integral Monitor" -EventId 1001 -EntryType Error -Message "Application health check failed: $_"
}
```

**Schedule Monitoring:**

```powershell
# Create scheduled task for monitoring
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-File C:\Scripts\MonitorIntegral.ps1"
$trigger = New-ScheduledTaskTrigger -Once -At (Get-Date) -RepetitionInterval (New-TimeSpan -Minutes 5)
Register-ScheduledTask -TaskName "Integral Health Monitor" -Action $action -Trigger $trigger -RunLevel Highest
```

### 5.5 Backup Strategy

**Database Backup:**

```powershell
# Create backup script
$backupPath = "C:\Backups\PostgreSQL"
$date = Get-Date -Format "yyyyMMdd_HHmmss"
$backupFile = "$backupPath\integral_db_$date.sql"

# Run PostgreSQL backup
& "C:\Program Files\PostgreSQL\16\bin\pg_dump.exe" -U integral_user -h localhost -d integral_db -f $backupFile
```

**Application Backup:**

- Regular backup of `C:\inetpub\IntegralBackend\` and `C:\inetpub\IntegralFrontend\`
- Backup configuration files
- Document application pool settings

---

## Testing and Verification

### 6.1 Test Backend API

**Health Check (HTTP):**

```powershell
# Test API health endpoint via custom domain
Invoke-WebRequest -Uri "http://api.integral.local:8080/health" -UseBasicParsing
```

**Health Check (HTTPS):**

```powershell
# Test HTTPS endpoint (may show certificate warning - this is normal for self-signed)
Invoke-WebRequest -Uri "https://api.integral.local:8443/health" -UseBasicParsing -SkipCertificateCheck
```

**API Endpoints:**

```powershell
# Test connectivity endpoint
Invoke-WebRequest -Uri "https://api.integral.local:8443/connectivity/ping" -UseBasicParsing -SkipCertificateCheck
```

### 6.2 Test Frontend Application

1. Open browser and navigate to `https://app.integral.local:8444`
2. **Accept the security warning** (self-signed certificate)
3. Verify login functionality works
4. Test authentication and authorization features
5. Check that API communication works properly

**For HTTP Testing (fallback):**

- Navigate to `http://app.integral.local:8081`

### 6.3 Distribute SSL Certificates to Client Computers

**For Network Users to Access Without Security Warnings:**

1. **Export Certificates from Server:**

```powershell
# Run on the server to export certificates
$apiCert = Get-ChildItem -Path "Cert:\LocalMachine\My" | Where-Object {$_.Subject -like "*api.integral.local*"}
$frontendCert = Get-ChildItem -Path "Cert:\LocalMachine\My" | Where-Object {$_.Subject -like "*app.integral.local*"}

# Export to files
Export-Certificate -Cert $apiCert -FilePath "C:\temp\api-integral-local.cer"
Export-Certificate -Cert $frontendCert -FilePath "C:\temp\app-integral-local.cer"

Write-Host "Certificates exported to C:\temp\"
```

1. **Install on Client Computers:**
   - Copy the `.cer` files to client computers
   - Double-click each certificate file
   - Click **Install Certificate**
   - Choose **Local Machine** → **Next**
   - Select **Place all certificates in the following store**
   - Click **Browse** → Select **Trusted Root Certification Authorities**
   - Click **Next** → **Finish**

### 6.3 Test Database Connectivity

```sql
-- Connect to PostgreSQL and verify
psql -U integral_user -d integral_db -h localhost

-- Check if tables exist
\dt

-- Exit
\q
```

---

## Troubleshooting Common Issues

### Issue 1: 500.30 ASP.NET Core app failed to start

**Solutions:**

- Check Event Viewer for detailed errors
- Verify .NET Runtime is installed
- Check file permissions
- Review appsettings.json syntax

### Issue 2: Database Connection Fails

**Solutions:**

- Verify PostgreSQL service is running
- Check connection string
- Verify user permissions
- Check firewall rules

### Issue 3: JWT Authentication Issues

**Solutions:**

- Verify JWT secrets match between frontend and backend
- Check token expiration settings
- Verify issuer/audience configuration

### Issue 4: Static Files Not Serving

**Solutions:**

- Verify `UseStaticFiles()` middleware is configured
- Check file permissions
- Ensure files are in wwwroot directory

---

## Access URLs Summary

After completing all steps, your Integral application will be accessible via:

### Local Server Access

**Frontend Application:**

- HTTP: `http://app.integral.local:8081`
- HTTPS: `https://app.integral.local:8444`

**Backend API:**

- HTTP: `http://api.integral.local:8080`
- HTTPS: `https://api.integral.local:8443`

### Network Access from Other Computers

1. **Add to client computers' hosts file** (`C:\Windows\System32\drivers\etc\hosts`):

   ```conf
   192.168.1.100    api.integral.local
   192.168.1.100    app.integral.local
   ```

   *(Replace 192.168.1.100 with your server's IP address)*

2. **Install SSL certificates** (optional, for HTTPS without warnings)
3. **Access the same URLs** as above from any network computer

### Database Connection (Server Only)

- **PostgreSQL:** localhost:5432 (internal use only)
- **Database:** integral_db
- **User:** integral_user

**Note:** Database is configured for local access only for security.

---

## Additional Recommendations

1. **Use a Reverse Proxy** (like IIS URL Rewrite) for better performance
2. **Implement SSL/TLS** for production environments
3. **Set up Log Rotation** to prevent disk space issues
4. **Monitor Performance** using Windows Performance Counters
5. **Regular Updates** for .NET runtime and PostgreSQL
6. **Backup Strategy** for both database and application files

This comprehensive guide provides you with all the steps needed to successfully host your Integral solution on Windows Server 2019 Standard using IIS and PostgreSQL. The deployment maintains your application's Clean Architecture principles while ensuring production-ready security and performance.
