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
Create: `Controllers/SuspectsController.cs`

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

## 11. Run the API

### Option 1: From Visual Studio
1. Open `TracePointInvestigations.sln`.
2. Set `TracePointAPI` as the startup project.
3. Press `F5`.
4. Swagger will open in your browser.

**Example:**
`https://localhost:7000/swagger`
The port may be different on your computer

*Note: The port may be different on your computer.*

### Option 2: From Command Line
```bash
cd TracePointAPI
dotnet restore
dotnet run
```

---

## 12. Create the React Application

1. Open **Command Prompt** or **Terminal**.
2. Navigate to your project folder:
   ```cmd
   cd C:\YourProjects\TracePointInvestigations
   ```
3. Create the React application:
   ```bash
   npx create-react-app tracepoint-client
   ```
4. Navigate into the React application:
   ```bash
   cd tracepoint-client
   ```

---

## 13. Install React Packages

1. Install **React Router**:
   ```bash
   npm install react-router-dom
   ```
2. Install **Axios**:
   ```bash
   npm install axios
   ```
3. Install **Bootstrap**:
   ```bash
   npm install bootstrap
   ```
4. Start the React application:
   ```bash
   npm start
   ```

The application will run at: `http://localhost:3000`

---

## 14. React Project Structure

Inside `src`, create the following structure:

```text
src/
├── components/
├── pages/
├── services/
└── __tests__/
```

### Folder Purposes

| Folder | Purpose |
| :--- | :--- |
| `components` | Reusable React components |
| `pages` | Application pages |
| `services` | API communication |
| `__tests__` | Frontend tests |

## 15. React API Service

Create: `src/services/api.js`

```javascript
import axios from 'axios';

const API_BASE_URL = 'https://localhost:7000/api';
// Change the port to match your API

const api = axios.create({
    baseURL: API_BASE_URL,
    headers: {
        'Content-Type': 'application/json',
    },
});

export const getCases = () => api.get('/cases');

export const getCaseById = (id) =>
    api.get(`/cases/${id}`);

export const getSuspects = () =>
    api.get('/suspects');

export const getSuspectById = (id) =>
    api.get(`/suspects/${id}`);

export const getEvidence = () =>
    api.get('/evidence');

export const getEvidenceById = (id) =>
    api.get(`/evidence/${id}`);

export const submitInvestigation = (investigation) =>
    api.post('/investigations', investigation);

export const getInvestigationSummary = () =>
    api.get('/investigations/summary');

export default api;
```

> ⚠️ **Important:** Check the URL displayed by Swagger and change `7000` if your API is running on another port.

---

## 16. Home Page

Create: `src/pages/Home.js`

```javascript
import React from 'react';
import { useNavigate } from 'react-router-dom';

function Home() {

    const navigate = useNavigate();

    return (
        <div className="text-center">

            <div className="py-5">

                <h1 className="home-title">
                    TRACEPOINT INVESTIGATIONS
                </h1>

                <div className="mt-4">

                    <span className="badge bg-danger fs-5">
                        ACTIVE CASE
                    </span>

                </div>

                <h2 className="mt-4 display-6">
                    THE MISSING PROTOTYPE
                </h2>

                <div className="row justify-content-center mt-4">

                    <div className="col-lg-8 col-md-10">

                        <p className="lead">

                            A prototype has disappeared from
                            a secure research laboratory.

                            <br />

                            Your task is to investigate the evidence
                            and identify the most likely suspect.

                        </p>

                    </div>

                </div>

                <button
                    className="btn btn-primary btn-lg px-5 py-3 mt-3"
                    onClick={() => navigate('/case')}
                >
                    START INVESTIGATION
                </button>

            </div>

        </div>
    );
}

export default Home;
```

---

## 17. Navigation Component

Create: `src/components/Navigation.js`

*Note: The final version includes a mobile hamburger menu.*

```javascript
import React, { useState } from 'react';
import { Link } from 'react-router-dom';

function Navigation() {

    const [isNavCollapsed, setIsNavCollapsed] =
        useState(true);

    const handleNavToggle = () => {
        setIsNavCollapsed(!isNavCollapsed);
    };

    return (

        <nav className="navbar navbar-expand-lg navbar-dark bg-dark sticky-top">

            <div className="container">

                <Link
                    className="navbar-brand fw-bold"
                    to="/"
                >
                    TracePoint
                </Link>

                <button
                    className="navbar-toggler"
                    type="button"
                    onClick={handleNavToggle}
                    aria-controls="navbarNav"
                    aria-expanded={!isNavCollapsed}
                    aria-label="Toggle navigation"
                >

                    <span className="navbar-toggler-icon"></span>

                </button>

                <div
                    className={`${isNavCollapsed ? 'collapse' : ''} navbar-collapse`}
                    id="navbarNav"
                >

                    <ul className="navbar-nav ms-auto">

                        <li className="nav-item">

                            <Link
                                className="nav-link"
                                to="/"
                                onClick={() =>
                                    setIsNavCollapsed(true)
                                }
                            >
                                HOME
                            </Link>

                        </li>

                        <li className="nav-item">

                            <Link
                                className="nav-link"
                                to="/case"
                                onClick={() =>
                                    setIsNavCollapsed(true)
                                }
                            >
                                CASE
                            </Link>

                        </li>

                        <li className="nav-item">

                            <Link
                                className="nav-link"
                                to="/suspects"
                                onClick={() =>
                                    setIsNavCollapsed(true)
                                }
                            >
                                SUSPECTS
                            </Link>

                        </li>

                        <li className="nav-item">

                            <Link
                                className="nav-link"
                                to="/evidence"
                                onClick={() =>
                                    setIsNavCollapsed(true)
                                }
                            >
                                EVIDENCE
                            </Link>

                        </li>

                        <li className="nav-item">

                            <Link
                                className="nav-link"
                                to="/investigation"
                                onClick={() =>
                                    setIsNavCollapsed(true)
                                }
                            >
                                INVESTIGATION
                            </Link>

                        </li>

                    </ul>

                </div>

            </div>

        </nav>
    );
}

export default Navigation;
```

---

## 18. App.js and Routing

*Note: The final App.js includes the navigation, responsive styling, and footer.*

Create/update: `src/App.js`

```javascript
import React from 'react';

import {
    BrowserRouter as Router,
    Routes,
    Route
} from 'react-router-dom';

import 'bootstrap/dist/css/bootstrap.min.css';

import './App.css';

import Navigation from './components/Navigation';

import Footer from './components/Footer';

import Home from './pages/Home';

import CasePage from './pages/CasePage';

import SuspectsPage from './pages/SuspectsPage';

import EvidencePage from './pages/EvidencePage';

import InvestigationPage from './pages/InvestigationPage';

function App() {

    return (

        <Router>

            <Navigation />

            <main className="container mt-4">

                <Routes>

                    <Route
                        path="/"
                        element={<Home />}
                    />

                    <Route
                        path="/case"
                        element={<CasePage />}
                    />

                    <Route
                        path="/suspects"
                        element={<SuspectsPage />}
                    />

                    <Route
                        path="/evidence"
                        element={<EvidencePage />}
                    />

                    <Route
                        path="/investigation"
                        element={<InvestigationPage />}
                    />

                </Routes>

            </main>

            <Footer />

        </Router>
    );
}

export default App;
```

---

## 19. Case Card

Create: `src/components/CaseCard.js`

```javascript
import React from 'react';

function CaseCard({ caseData }) {

    if (!caseData) {

        return (
            <div className="alert alert-warning">
                No case data available
            </div>
        );
    }
}
```
## 20. Case Page

Create/update: `src/pages/CasePage.js`

```javascript
import React, {
    useState,
    useEffect
} from 'react';

import { useNavigate } from 'react-router-dom';

import { getCases } from '../services/api';

import CaseCard from '../components/CaseCard';

function CasePage() {

    const [caseData, setCaseData] =
        useState(null);

    const [loading, setLoading] =
        useState(true);

    const [error, setError] =
        useState(null);

    const navigate = useNavigate();

    useEffect(() => {

        const fetchCase = async () => {

            try {

                const response = await getCases();

                // Assuming the first case is the active case
                if (
                    response.data &&
                    response.data.length > 0
                ) {

                    setCaseData(response.data[0]);

                } else {

                    setError('No case found');

                }

            } catch (err) {

                setError(
                    'Failed to load case data. Please ensure the API is running.'
                );

                console.error(
                    'Error fetching case:',
                    err
                );

            } finally {

                setLoading(false);

            }
        };

        fetchCase();

    }, []);

    if (loading) {

        return (

            <div className="text-center mt-5">

                <div
                    className="spinner-border text-primary"
                    role="status"
                >

                    <span className="visually-hidden">
                        Loading...
                    </span>

                </div>

                <p className="mt-2">
                    Loading case data...
                </p>

            </div>
        );
    }

    if (error) {

        return (

            <div
                className="alert alert-danger mt-3"
                role="alert"
            >
                {error}
            </div>
        );
    }

    return (

        <div>

            <h2 className="text-center mb-4">
                Case Details
            </h2>

            <CaseCard caseData={caseData} />

            <div className="d-flex justify-content-center gap-3 mt-4">

                <button
                    className="btn btn-primary btn-lg"
                    onClick={() =>
                        navigate('/suspects')
                    }
                >
                    View Suspects
                </button>

                <button
                    className="btn btn-secondary btn-lg"
                    onClick={() =>
                        navigate('/evidence')
                    }
                >
                    View Evidence
                </button>

            </div>

        </div>
    );
}

export default CasePage;
```

---

## 21. Suspect Card

Create: `src/components/SuspectCard.js`

```javascript
import React from 'react';

function SuspectCard({
    suspect,
    onSelect,
    isSelected
}) {

    return (

        <div
            className={`card h-100 shadow-sm ${
                isSelected
                    ? 'border-primary border-3'
                    : ''
            }`}
            onClick={() => onSelect(suspect)}
            style={{ cursor: 'pointer' }}
        >

            <div className="card-body">

                <h5 className="card-title">
                    {suspect.name}
                </h5>

                <h6 className="card-subtitle mb-2 text-muted">
                    {suspect.occupation}
                </h6>

                <p className="card-text">
                    {suspect.description}
                </p>

                {isSelected && (

                    <span className="badge bg-primary">
                        Selected
                    </span>

                )}

            </div>

        </div>
    );
}

export default SuspectCard;
```

---

## 22. Suspects Page

Create/update: `src/pages/SuspectsPage.js`

```javascript
import React, {
    useState,
    useEffect
} from 'react';

import { getSuspects } from '../services/api';

import SuspectCard from '../components/SuspectCard';

function SuspectsPage() {

    const [suspects, setSuspects] =
        useState([]);

    const [loading, setLoading] =
        useState(true);

    const [error, setError] =
        useState(null);

    const [selectedSuspect, setSelectedSuspect] =
        useState(null);

    useEffect(() => {

        const fetchSuspects = async () => {

            try {

                const response =
                    await getSuspects();

                setSuspects(response.data);

            } catch (err) {

                setError(
                    'Failed to load suspects. Please ensure the API is running.'
                );

                console.error(
                    'Error fetching suspects:',
                    err
                );

            } finally {

                setLoading(false);

            }
        };

        fetchSuspects();

    }, []);

    const handleSelectSuspect = (suspect) => {

        setSelectedSuspect(suspect);

    };

    if (loading) {

        return (

            <div className="text-center mt-5">

                <div
                    className="spinner-border text-primary"
                    role="status"
                >

                    <span className="visually-hidden">
                        Loading...
                    </span>

                </div>

                <p className="mt-2">
                    Loading suspects...
                </p>

            </div>
        );
    }

    if (error) {

        return (

            <div
                className="alert alert-danger mt-3"
                role="alert"
            >
                {error}
            </div>
        );
    }

    return (

        <div>

            <h2 className="text-center mb-4">
                Suspects
            </h2>

            {selectedSuspect && (

                <div className="alert alert-info">

                    <strong>
                        Selected Suspect:
                    </strong>{' '}

                    {selectedSuspect.name}

                    <button
                        className="btn btn-sm btn-outline-secondary ms-3"
                        onClick={() =>
                            setSelectedSuspect(null)
                        }
                    >
                        Clear Selection
                    </button>

                </div>

            )}

            <div className="row row-cols-1 row-cols-md-3 g-4">

                {suspects.map((suspect) => (

                    <div
                        className="col"
                        key={suspect.suspectID}
                    >

                        <SuspectCard
                            suspect={suspect}
                            onSelect={handleSelectSuspect}
                            isSelected={
                                selectedSuspect?.suspectID ===
                                suspect.suspectID
                            }
                        />

                    </div>

                ))}

            </div>

            <div className="text-center mt-4">

                <button
                    className="btn btn-primary"
                    onClick={() =>
                        window.location.href =
                            '/investigation'
                    }
                >
                    Proceed to Investigation
                </button>

            </div>

        </div>
    );
}

export default SuspectsPage;
```

---

## 23. Evidence Card

Create: `src/components/EvidenceCard.js`

```javascript
import React, { useState } from 'react';

function EvidenceCard({
    evidence,
    onExamine
}) {

    const [isExpanded, setIsExpanded] =
        useState(false);

    const handleExamine = () => {

        setIsExpanded(!isExpanded);

        if (onExamine) {

            onExamine(evidence);

        }
    };

    return (

        <div className="card h-100 shadow-sm">

            <div className="card-body">

                <h5 className="card-title">
{evidence.title}
                </h5>

                <p className="card-text text-muted">

                    <small>
                        Location: {evidence.location}
                    </small>

                </p>

                {isExpanded && (

                    <div className="mt-3 p-3 bg-light rounded">

                        <h6>
                            Evidence Details
                        </h6>

                        <p>
                            {evidence.description}
                        </p>

                    </div>

                )}

                <button
                    className="btn btn-outline-primary mt-2"
                    onClick={handleExamine}
                >
                    {
                        isExpanded
                            ? 'Hide Evidence'
                            : 'Examine Evidence'
                    }
                </button>

            </div>

        </div>
    );
}

export default EvidenceCard;
```
## 24. Evidence Page
Create/update: `src/pages/EvidencePage.js`

```javascript
import React, {
    useState,
    useEffect
} from 'react';

import { getEvidence } from '../services/api';

import EvidenceCard
    from '../components/EvidenceCard';

function EvidencePage() {

    const [evidence, setEvidence] =
        useState([]);

    const [loading, setLoading] =
        useState(true);

    const [error, setError] =
        useState(null);

    const [selectedEvidence, setSelectedEvidence] =
        useState(null);

    useEffect(() => {

        const fetchEvidence = async () => {

            try {

                const response =
                    await getEvidence();

                setEvidence(response.data);

            } catch (err) {

                setError(
                    'Failed to load evidence. Please ensure the API is running.'
                );

                console.error(
                    'Error fetching evidence:',
                    err
                );

            } finally {

                setLoading(false);

            }
        };

        fetchEvidence();

    }, []);

    const handleExamineEvidence =
        (evidenceItem) => {

            setSelectedEvidence(
                evidenceItem
            );

        };

    if (loading) {

        return (

            <div className="text-center mt-5">

                <div
                    className="spinner-border text-primary"
                    role="status"
                >

                    <span className="visually-hidden">
                        Loading...
                    </span>

                </div>

                <p className="mt-2">
                    Loading evidence...
                </p>

            </div>
        );
    }

    if (error) {

        return (

            <div
                className="alert alert-danger mt-3"
                role="alert"
            >
                {error}
            </div>
        );
    }

    return (

        <div>

            <h2 className="text-center mb-4">
                Evidence
            </h2>

            {selectedEvidence && (

                <div className="alert alert-info">

                    <strong>
                        Currently Examining:
                    </strong>{' '}

                    {selectedEvidence.title}

                    <br />

                    <small>
                        {selectedEvidence.description}
                    </small>

                    <button
                        className="btn btn-sm btn-outline-secondary ms-3"
                        onClick={() =>
                            setSelectedEvidence(null)
                        }
                    >
                        Clear Selection
                    </button>

                </div>

            )}

            <div className="row row-cols-1 row-cols-md-2 row-cols-xl-3 g-4">

                {evidence.map((item) => (

                    <div
                        className="col"
                        key={item.evidenceID}
                    >

                        <EvidenceCard
                            evidence={item}
                            onExamine={
                                handleExamineEvidence
                            }
                        />

                    </div>

                ))}

            </div>

            <div className="text-center mt-4">

                <button
                    className="btn btn-success"
                    onClick={() =>
                        window.location.href =
                            '/investigation'
                    }
                >
                    Proceed to Investigation
                </button>

            </div>

        </div>
    );
}

export default EvidencePage;
```
## 25.Investigation Page
Create/update: `src/pages/InvestigationPage.js`

```javascript
import React, {
    useState,
    useEffect
} from 'react';

import { useNavigate } from 'react-router-dom';

import {
    getSuspects,
    submitInvestigation
} from '../services/api';

function InvestigationPage() {

    const [suspects, setSuspects] =
        useState([]);

    const [selectedSuspectId, setSelectedSuspectId] =
        useState('');

    const [conclusion, setConclusion] =
        useState('');

    const [loading, setLoading] =
        useState(false);

    const [submitted, setSubmitted] =
        useState(false);

    const [error, setError] =
        useState(null);

    const [validationError, setValidationError] =
        useState('');

    const navigate = useNavigate();

    // Fetch suspects when page loads
    useEffect(() => {

        const fetchSuspects = async () => {

            try {

                const response =
                    await getSuspects();

                setSuspects(response.data);

            } catch (err) {

                setError(
                    'Failed to load suspects. Please ensure the API is running.'
                );

                console.error(
                    'Error fetching suspects:',
                    err
                );

            }
        };

        fetchSuspects();

    }, []);

    const handleSubmit = async (e) => {

        e.preventDefault();

        setValidationError('');

        // Validation: Check if suspect is selected
        if (!selectedSuspectId) {

            setValidationError(
                'Please select a suspect.'
            );

            return;
        }

        // Validation: Check if conclusion is entered
        if (!conclusion.trim()) {

            setValidationError(
                'Please enter your investigation conclusion.'
            );

            return;
        }

        setLoading(true);

        try {

            const investigationData = {

                caseID: 1,

                suspectID:
                    parseInt(selectedSuspectId),

                conclusion:
                    conclusion.trim(),

                dateStarted:
                    new Date().toISOString()

            };

            const response =
                await submitInvestigation(
                    investigationData
                );

            console.log(
                'Investigation submitted:',
                response.data
            );

            setSubmitted(true);

            setLoading(false);

        } catch (err) {

            setError(
                'Failed to submit investigation. Please try again.'
            );

            console.error(
                'Error submitting investigation:',
                err
            );

            setLoading(false);
        }
    };

    const handleReset = () => {

        setSelectedSuspectId('');

        setConclusion('');

        setSubmitted(false);

        setError(null);

        setValidationError('');
    };

    if (submitted) {

        return (

            <div className="text-center mt-5">

                <div
                    className="alert alert-success"
                    role="alert"
                >

                    <h4 className="alert-heading">
                        Investigation Submitted Successfully
                    </h4>

                    <p>
                        Your investigation has been
                        recorded by TracePoint Investigations.
                    </p>

                    <hr />

                    <p className="mb-0">

                        <strong>
                            Selected Suspect:
                        </strong>{' '}

                        {
                            suspects.find(
                                s =>
                                    s.suspectID ===
                                    parseInt(selectedSuspectId)
                            )?.name
                        }

                    </p>

                    <p className="mb-3">

                        <strong>
                            Conclusion:
                        </strong>{' '}

                        {conclusion}

                    </p>

                    <button
                        className="btn btn-primary"
                        onClick={handleReset}
                    >
                        Start New Investigation
                    </button>

                </div>

            </div>
        );
    }

    if (error) {

        return (

            <div
                className="alert alert-danger mt-3"
                role="alert"
            >

                {error}

                <button
                    className="btn btn-sm btn-outline-danger ms-3"
                    onClick={() => setError(null)}
                >
                    Dismiss
                </button>

            </div>
        );
    }

    return (

        <div>

            <h2 className="text-center mb-4">
                Submit Investigation
            </h2>

            {validationError && (

                <div
                    className="alert alert-warning"
                    role="alert"
                >
                    {validationError}
                </div>

            )}

            <form onSubmit={handleSubmit}>

                {/* Suspect Selection */}

                <div className="mb-4">

                    <label
                        htmlFor="suspectSelect"
                        className="form-label fw-bold"
                    >
                        Select Suspect
                    </label>

                    <select
                        id="suspectSelect"
                        className="form-select form-select-lg"
                        value={selectedSuspectId}
                        onChange={(e) =>
                            setSelectedSuspectId(
                                e.target.value
                            )
                        }
                    >

                        <option value="">
                            -- Select a suspect --
                        </option>

                        {suspects.map((suspect) => (

                            <option
                                key={suspect.suspectID}
                                value={suspect.suspectID}
                            >
                                {suspect.name} -
                                {' '}
                                {suspect.occupation}
                            </option>

                        ))}

                    </select>

                    {selectedSuspectId && (

                        <div className="mt-2 text-success">

                            <small>
                                Selected:{' '}

                                {
                                    suspects.find(
                                        s =>
                                            s.suspectID ===
                                            parseInt(
                                                selectedSuspectId
                                            )
                                    )?.name
                                }

                            </small>

                        </div>

                    )}

                </div>

                {/* Conclusion Text Area */}

                <div className="mb-4">

                    <label
                        htmlFor="conclusionText"
                        className="form-label fw-bold"
                    >
                        Investigation Conclusion
                    </label>

                    <textarea
                        id="conclusionText"
                        className="form-control"
                        rows="5"
                        placeholder="Explain your conclusion. Example: Jamie Smith appears to be the most likely suspect because his access card was used to enter the laboratory shortly before the prototype disappeared."
                        value={conclusion}
                        onChange={(e) =>
                            setConclusion(
                                e.target.value
                            )
                        }
                    ></textarea>

                    <div className="mt-1 text-muted">

                        <small>
                            {conclusion.length}
                            {' '}
                            characters
                        </small>

                    </div>

                </div>

                {/* Submit Button */}

                <div className="d-flex gap-3">

                    <button
                        type="submit"
                        className="btn btn-success btn-lg flex-grow-1"
                        disabled={loading}
                    >

                        {loading ? (

                            <>

                                <span
                                    className="spinner-border spinner-border-sm me-2"
                                    role="status"
                                ></span>

                                Submitting...

                            </>

                        ) : (

                            'SUBMIT INVESTIGATION'

                        )}

                    </button>

                    <button
                        type="button"
                        className="btn btn-outline-secondary"
                        onClick={() =>
                            navigate('/case')
                        }
                    >
                        Cancel
                    </button>

                </div>

            </form>

        </div>
    );
}

export default InvestigationPage;
```

## 26. Footer Component
Create: `src/components/Footer.js`

```javascript
import React from 'react';

function Footer() {

    return (

        <footer className="footer mt-5">

            <div className="container">

                <p className="mb-0">

                    &copy; {new Date().getFullYear()}
                    {' '}
                    TracePoint Investigations.
                    All rights reserved.

                </p>

                <small className="text-muted">

                    Operation Digital Detective -
                    Full Stack Web Application

                </small>

            </div>

        </footer>
    );
}

export default Footer;
```

## 27. Custom CSS

Create: `src/App.css`

```css
/* Global Styles */
body {
    background-color: #f8f9fa;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    min-height: 100vh;
}

/* Container padding for mobile */
.container {
    padding-left: 15px;
    padding-right: 15px;
}

/* Card hover effects */
.card {
    transition: transform 0.2s ease-in-out, box-shadow 0.2s ease-in-out;
}

.card:hover {
    transform: translateY(-5px);
    box-shadow: 0 10px 20px rgba(0, 0, 0, 0.15) !important;
}

/* Home page styling */
.home-title {
    font-size: 3.5rem;
    font-weight: 900;
    letter-spacing: 2px;
    background: linear-gradient(135deg, #1a1a2e, #16213e);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
}

@media (max-width: 768px) {
    .home-title {
        font-size: 2.2rem;
    }
}

/* Evidence card expand animation */
.evidence-details {
    animation: fadeIn 0.3s ease-in-out;
}

@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateY(-10px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

/* Status badges */
.badge-status {
    font-size: 0.9rem;
    padding: 0.5rem 1rem;
    border-radius: 20px;
}

/* Responsive button groups */
.btn-group-responsive {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    justify-content: center;
}

@media (max-width: 576px) {
    .btn-group-responsive .btn {
        width: 100%;
        margin-bottom: 5px;
    }
}

/* Form styling */
textarea.form-control {
    resize: vertical;
    min-height: 120px;
}

/* Success message animation */
.success-alert {
    animation: slideDown 0.5s ease-in-out;
}

@keyframes slideDown {
    from {
        opacity: 0;
        transform: translateY(-30px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

/* Footer */
.footer {
    margin-top: 50px;
    padding: 20px 0;
    background-color: #1a1a2e;
    color: #fff;
    text-align: center;
}

/* Mobile card adjustments */
@media (max-width: 576px) {
    .card-body {
        padding: 1rem;
    }
    .card-title {
        font-size: 1.1rem;
    }
    .display-3 {
        font-size: 2.5rem;
    }
}

/* Tablet adjustments */
@media (min-width: 768px) and (max-width: 992px) {
    .container {
        max-width: 720px;
    }
    .row-cols-md-3 > .col {
        flex: 0 0 50%;
        max-width: 50%;
    }
}
```

---

## 28. Import CSS in App.js

At the top of `App.js`, include:

```javascript
import './App.css';
```

---

## 29. Testing Packages

### 29.1 Frontend Testing Packages
Open a terminal inside `tracepoint-client` and run:

```bash
npm install --save-dev @testing-library/user-event @testing-library/jest-dom
```

### 29.2 Backend Testing Packages
Create an xUnit test project named `TracePointAPI.Tests` and install the following packages:

```powershell
Install-Package Microsoft.EntityFrameworkCore.InMemory
Install-Package Microsoft.AspNetCore.Mvc.Testing
Install-Package System.Net.Http.Json
```

---

## 30. Frontend Unit Tests

Create: `src/__tests__/InvestigationForm.test.js`

```javascript
import React from 'react';

import {
    render,
    screen,
    fireEvent,
    waitFor
} from '@testing-library/react';

import {
    BrowserRouter
} from 'react-router-dom';

import {
    getSuspects,
    submitInvestigation
} from '../services/api';

import InvestigationPage
    from '../pages/InvestigationPage';

jest.mock('../services/api');

const mockSuspects = [
    {
        suspectID: 1,
        name: 'Alex Morgan',
        occupation: 'Software Developer',
        description: 'Test description'
    },
    {
        suspectID: 2,
        name: 'Jamie Smith',
        occupation: 'Security Officer',
        description: 'Test description'
    },
    {
        suspectID: 3,
        name: 'Taylor Williams',
        occupation: 'Research Assistant',
        description: 'Test description'
    }
];

describe(
    'Investigation Form Unit Tests',
    () => {

        beforeEach(() => {
            jest.clearAllMocks();

            getSuspects.mockResolvedValue({
                data: mockSuspects
            });

            submitInvestigation.mockResolvedValue({
                data: {
                    investigationID: 1
                }
            });
        });

        test(
            'shows validation error when no suspect is selected',
            async () => {
                render(
                    <BrowserRouter>
                        <InvestigationPage />
                    </BrowserRouter>
                );

                await waitFor(() => {
                    expect(getSuspects).toHaveBeenCalled();
                });

                const submitButton = screen.getByText('SUBMIT INVESTIGATION');
                fireEvent.click(submitButton);

                const errorMessage = await screen.findByText('Please select a suspect.');
                expect(errorMessage).toBeInTheDocument();
                expect(submitInvestigation).not.toHaveBeenCalled();
            }
        );

        test(
            'shows validation error when conclusion is empty',
            async () => {
                render(
                    <BrowserRouter>
                        <InvestigationPage />
                    </BrowserRouter>
                );

                await waitFor(() => {
                    expect(getSuspects).toHaveBeenCalled();
                });

                const select = screen.getByLabelText('Select Suspect');
                fireEvent.change(select, { target: { value: '1' } });

                const submitButton = screen.getByText('SUBMIT INVESTIGATION');
                fireEvent.click(submitButton);

                const errorMessage = await screen.findByText('Please enter your investigation conclusion.');
                expect(errorMessage).toBeInTheDocument();
                expect(submitInvestigation).not.toHaveBeenCalled();
            }
        );

        test(
            'successfully submits investigation when all fields are valid',
            async () => {
                render(
                    <BrowserRouter>
                        <InvestigationPage />
                    </BrowserRouter>
                );

                await waitFor(() => {
                    expect(getSuspects).toHaveBeenCalled();
                });

                const select = screen.getByLabelText('Select Suspect');
                fireEvent.change(select, { target: { value: '1' } });

                const textarea = screen.getByLabelText('Investigation Conclusion');
  fireEvent.change(
                    textarea,
                    {
                        target: {
                            value:
                                'This is a valid conclusion.'
                        }
                    }
                );

                const submitButton =
                    screen.getByText(
                        'SUBMIT INVESTIGATION'
                    );

                fireEvent.click(
                    submitButton
                );

                await waitFor(() => {

                    expect(
                        submitInvestigation
                    ).toHaveBeenCalled();

                    expect(
                        submitInvestigation
                    ).toHaveBeenCalledWith(

                        expect.objectContaining({

                            caseID: 1,

                            suspectID: 1,

                            conclusion:
                                'This is a valid conclusion.'

                        })

                    );

                });

                const successMessage =
                    await screen.findByText(
                        'Investigation Submitted Successfully'
                    );

                expect(
                    successMessage
                ).toBeInTheDocument();

            }
        );

    }
);
               
```
