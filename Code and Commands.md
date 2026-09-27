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
```

---

## 4. Backend Models

Create a folder called: `Models`

### 4.1 Case.cs
```csharp
namespace TracePointAPI.Models 
{ 
    public class Case 
    { 
        public int CaseID { get; set; } 
        public string CaseName { get; set; } = string.Empty; 
        public string Description { get; set; } = string.Empty; 
        public string Status { get; set; } = string.Empty; 
    } 
}
```

### 4.2 Suspect.cs
```csharp
namespace TracePointAPI.Models
{
    public class Suspect
    {
        public int SuspectID { get; set; }
        public string Name { get; set; } = string.Empty;
        public string Occupation { get; set; } = string.Empty;
        public string Description { get; set; } = string.Empty;
    }
}
```

### 4.3 Evidence.cs
```csharp
namespace TracePointAPI.Models
{
    public class Evidence
    {
        public int EvidenceID { get; set; }
        public string Title { get; set; } = string.Empty;
        public string Description { get; set; } = string.Empty;
        public string Location { get; set; } = string.Empty;
    }
}
```

### 4.4 Investigation.cs
```csharp
namespace TracePointAPI.Models
{
    public class Investigation
    {
        public int InvestigationID { get; set; }
        public int CaseID { get; set; }
        public string Description { get; set; } = string.Empty;
        public string Date { get; set; } = string.Empty;
    }
}
```

## 5. Database Setup

Create a folder called: 'Data'
Create: 'ApplicationDbContext.cs'

### 5.1 ApplicationDbContext.cs

```bash
using Microsoft.EntityFrameworkCore;
using TracePointAPI.Models;

namespace TracePointAPI.Data
{
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
            : base(options)
        {
        }

        public DbSet<Case> Cases { get; set; }

        public DbSet<Suspect> Suspects { get; set; }

        public DbSet<Evidence> Evidences { get; set; }

        public DbSet<Investigation> Investigations { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);

            // Seed Cases
            modelBuilder.Entity<Case>().HasData(
                new Case
                {
                    CaseID = 1,
                    CaseName = "The Missing Prototype",
                    Description = "A technology prototype has disappeared from the research laboratory between 22:00 and 00:00.",
                    Status = "Open"
                }
            );

            // Seed Suspects
            modelBuilder.Entity<Suspect>().HasData(
                new Suspect
                {
                    SuspectID = 1,
                    Name = "Alex Morgan",
                    Occupation = "Software Developer",
                    Description = "Alex developed the software used by the prototype and had access to the laboratory."
                },

                new Suspect
                {
                    SuspectID = 2,
                    Name = "Jamie Smith",
                    Occupation = "Security Officer",
                    Description = "Jamie was responsible for security at the building on the night of the incident."
                },

                new Suspect
                {
                    SuspectID = 3,
                    Name = "Taylor Williams",
                    Occupation = "Research Assistant",
                    Description = "Taylor worked with the research team and had access to the laboratory during working hours."
                }
            );

            // Seed Evidence
            modelBuilder.Entity<Evidence>().HasData(
                new Evidence
                {
                    EvidenceID = 1,
                    Title = "Security Access Log",
                    Description = "Jamie Smith's access card was used to enter the research laboratory at 23:41.",
                    Location = "Security Office"
                },

                new Evidence
                {
                    EvidenceID = 2,
                    Title = "CCTV Report",
                    Description = "CCTV footage shows a person entering the laboratory at approximately 23:43. The person's face cannot be clearly identified.",
                    Location = "Research Laboratory"
                },

                new Evidence
                {
                    EvidenceID = 3,
                    Title = "Fingerprint Report",
                    Description = "A partial fingerprint was found on the prototype storage cabinet. The fingerprint belongs to a person who regularly works in the laboratory.",
                    Location = "Research Laboratory"
                },

                new Evidence
                {
                    EvidenceID = 4,
                    Title = "Email Message",
                    Description = "An email sent shortly before the incident states: 'The prototype must be moved before tomorrow's demonstration.'",
                    Location = "Archive Room"
                },

                new Evidence
                {
                    EvidenceID = 5,
                    Title = "Photograph",
                    Description = "A photograph taken after the incident shows that the prototype cabinet was open and the laboratory lights were switched off.",
                    Location = "Research Laboratory"
                }
            );
        }
    }
}
```

## 6. Database Connection

Open: 'appsettings.json'

Replace the contents with:

```bash
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },

  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=TracePointDB;Trusted_Connection=True;MultipleActiveResultSets=true;TrustServerCertificate=True"
  },

  "AllowedHosts": "*"
}
```
## 7. Configure Program.cs
 Use the following:

 ```bash
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },

  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=TracePointDB;Trusted_Connection=True;MultipleActiveResultSets=true;TrustServerCertificate=True"
  },

  "AllowedHosts": "*"
}
```
## 8. Create and Apply the Database Migration

Follow this menu path to open the console:

```text
[ Tools ]
    │
    ▼
[ NuGet Package Manager ]
    │
    ▼
[ Package Manager Console ]
```

Run the following commands in order:

```powershell
Add-Migration InitialCreate
Update-Database
```

### Database Information

The database will be created as: 
`TracePointDB`

The database will contain the following tables:
- Cases
- Suspects
- Evidences
- Investigations

## 9. API Controllers

### 9.1 CasesController.cs
Create: 'Controllers/CasesController.cs'

```bash
## 8. Create and Apply the Database Migration

Follow this menu path to open the console:

```text
[ Tools ]
    │
    ▼
[ NuGet Package Manager ]
    │
    ▼
[ Package Manager Console ]
```

Run the following commands in order:

```powershell
Add-Migration InitialCreate
Update-Database
```

### 9.1 Database Information

```bash
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using TracePointAPI.Data;
using TracePointAPI.Models;

namespace TracePointAPI.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class CasesController : ControllerBase
    {
        private readonly ApplicationDbContext _context;

        public CasesController(ApplicationDbContext context)
        {
            _context = context;
        }

        // GET: api/cases
        [HttpGet]
        public async Task<ActionResult<IEnumerable<Case>>> GetCases()
        {
            return await _context.Cases.ToListAsync();
        }

        // GET: api/cases/{id:int}
        [HttpGet("{id:int}")]
        public async Task<ActionResult<Case>> GetCase(int id)
        {
            var caseItem = await _context.Cases.FindAsync(id);

            if (caseItem == null)
            {
                return NotFound();
            }

            return caseItem;
        }
    }
}
```
### 9.2 SuspectController.cs
Create: 'Controllers/SuspectsController.cs`

```bash
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using TracePointAPI.Data;
using TracePointAPI.Models;

namespace TracePointAPI.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class SuspectsController : ControllerBase
    {
        private readonly ApplicationDbContext _context;

        public SuspectsController(ApplicationDbContext context)
        {
            _context = context;
        }

        // GET: api/suspects
        [HttpGet]
        public async Task<ActionResult<IEnumerable<Suspect>>> GetSuspects()
        {
            return await _context.Suspects.ToListAsync();
        }

        // GET: api/suspects/{id:int}
        [HttpGet("{id:int}")]
        public async Task<ActionResult<Suspect>> GetSuspect(int id)
        {
            var suspect = await _context.Suspects.FindAsync(id);

            if (suspect == null)
            {
                return NotFound();
            }

            return suspect;
        }
    }
}
```



