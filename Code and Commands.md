# Operation Digital Detective - TracePoint Investigations

## 1. Installation and Setup

### 1.1 Prerequisites

Before starting, make sure the following are installed:

| Software | Requirement |
|---|---|
| Visual Studio | 2022 or later |
| .NET | .NET 8.0 SDK |
| Database | SQL Server Express LocalDB |
| Code Editor | Visual Studio Code |
| JavaScript Runtime | Node.js and npm |

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
```bash
using System;

namespace TracePointAPI.Models
{
    public class Investigation
    {
        public int InvestigationID { get; set; }

        public int CaseID { get; set; }

        public int SuspectID { get; set; }

        public string Conclusion { get; set; } = string.Empty;

        public DateTime DateStarted { get; set; }
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
using Microsoft.EntityFrameworkCore;
using TracePointAPI.Data;

var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Add CORS policy
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowReactApp",
        policy =>
        {
            policy.WithOrigins("http://localhost:3000")
                  .AllowAnyHeader()
                  .AllowAnyMethod();
        });
});

// Register DbContext
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString(
            "DefaultConnection")));

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

// Use CORS
app.UseCors("AllowReactApp");

app.UseAuthorization();

app.MapControllers();

app.Run();
```

**Why is CORS required?**
The React frontend runs on:

```bash
http://localhost:3000
```
while the ASP.NET Core API runs on a different port
The CORS policy allows the React application to communicate with the API.

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
- Cases - Stores case information
- Suspects - Stores suspect information
- Evidences - Stores evidence information
- Investigations - Stores submitted investigations

## 9. API Controllers
Controllers handle requests from the React frontend and return data as JSON

### 9.1 CasesController.cs
Create: `Controllers/CasesController.cs`

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

### 9.3 EvidenceController.cs
Create: `Controllers/EvidenceController.cs`

```bash
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using TracePointAPI.Data;
using TracePointAPI.Models;

namespace TracePointAPI.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class EvidenceController : ControllerBase
    {
        private readonly ApplicationDbContext _context;

        public EvidenceController(ApplicationDbContext context)
        {
            _context = context;
        }

        // GET: api/evidence
        [HttpGet]
        public async Task<ActionResult<IEnumerable<Evidence>>> GetEvidence()
        {
            return await _context.Evidences.ToListAsync();
        }

        // GET: api/evidence/{id:int}
        [HttpGet("{id:int}")]
        public async Task<ActionResult<Evidence>> GetEvidence(int id)
        {
            var evidence = await _context.Evidences.FindAsync(id);

            if (evidence == null)
            {
                return NotFound();
            }

            return evidence;
        }
    }
}
```

### 9.4 InvestigationsController.cs

Create: `Controllers/InvestigationsController.cs`

```bash
using Microsoft.AspNetCore.Mvc;
using TracePointAPI.Data;
using TracePointAPI.Models;
using Dapper;
using Microsoft.Data.SqlClient;
using Microsoft.EntityFrameworkCore;

namespace TracePointAPI.Controllers
{
    [Route("api/[controller]")]
    [ApiController]
    public class InvestigationsController : ControllerBase
    {
        private readonly ApplicationDbContext _context;
        private readonly IConfiguration _configuration;

        public InvestigationsController(
            ApplicationDbContext context,
            IConfiguration configuration)
        {
            _context = context;
            _configuration = configuration;
        }

        // POST: api/investigations
        [HttpPost]
        public async Task<ActionResult<Investigation>>
            PostInvestigation(Investigation investigation)
        {
            // Set the date to now
            investigation.DateStarted = DateTime.Now;

            _context.Investigations.Add(investigation);

            await _context.SaveChangesAsync();

            return CreatedAtAction(
                nameof(GetInvestigation),
                new { id = investigation.InvestigationID },
                investigation);
        }

        // GET: api/investigations/{id}
        [HttpGet("{id}")]
        public async Task<ActionResult<Investigation>>
            GetInvestigation(int id)
        {
            var investigation =
                await _context.Investigations.FindAsync(id);

            if (investigation == null)
            {
                return NotFound();
            }

            return investigation;
        }

        // GET: api/investigations/summary
        // DAPPER IMPLEMENTATION
        // Retrieves investigations with suspect names
        [HttpGet("summary")]
        public async Task<ActionResult<IEnumerable<object>>>
            GetInvestigationSummary()
        {
            using var connection = new SqlConnection(
                _configuration.GetConnectionString(
                    "DefaultConnection"));

            var query = @"
                SELECT
                    i.InvestigationID,
                    s.Name AS SuspectName,
                    i.Conclusion,
                    i.DateStarted
                FROM Investigations i
                INNER JOIN Suspects s
                    ON i.SuspectID = s.SuspectID
                ORDER BY i.DateStarted DESC";

            var investigations =
                await connection.QueryAsync(query);

            return Ok(investigations);
        }
    }
}
```
## 10. API Endpints

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| GET | `/api/cases` | Get all cases |
| GET | `/api/cases/{id:int}` | Get case by ID |
| GET | `/api/suspects` | Get all suspects |
| GET | `/api/suspects/{id:int}` | Get suspect by ID |
| GET | `/api/evidence` | Get all evidence |
| GET | `/api/evidence/{id:int}` | Get evidence by ID |
| POST | `/api/investigations` | Submit a new investigation |
| GET | `/api/investigations/summary` | Get investigations with suspect names using Dapper |



