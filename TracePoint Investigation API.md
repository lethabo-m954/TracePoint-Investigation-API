# Operation Digital Detective - TracePoint Investigations

## 1. Installation and Setup

### 1.1 Prerequisites

Before starting, make sure the following are installed:

- Visual Studio 2022 or later
- .NET 8.0 SDK
- SQL Server Express LocalDB
- Visual Studio Code
- Node.js and npm

---

## 2. Create the ASP.NET Core Web API

Open Visual Studio 2022.

Go to:

**File → New → Project**

Search for:

`ASP.NET Core Web API`

Use the following settings:

| Setting | Value |
|---|---|
| Project Name | `TracePointAPI` |
| Solution Name | `TracePointInvestigations` |
| Framework | `.NET 8.0` |
| Authentication | `None` |
| Configure for HTTPS | Checked |
| Enable OpenAPI | Checked |
| Use Controllers | Checked |

---

## 3. Install Backend Packages

Open:

**Tools → NuGet Package Manager → Package Manager Console**

Run the following commands one at a time:

```powershell
Install-Package Microsoft.EntityFrameworkCore.SqlServer
Install-Package Microsoft.EntityFrameworkCore.Tools
Install-Package Dapper
Install-Package Microsoft.EntityFrameworkCore.Design
