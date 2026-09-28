** 1. TracePoint-Investigation-API - Digital Investigation System

A full-stack investigation app for TracePoint Investigations. ASP.NET Core Web API + Entity Framework Core + Dapper backend with a React (React Router + Bootstrap) frontend. Investigators view a case, examine suspects and evidence, select a suspect, submit a conclusion, and persist the investigation to SQL Server.

** 2. Application Description

TracePoint Investigations is a fictional private investigation company. This
system allows an investigator to work through a digital case file for
The Missing Prototype — a case in which an experimental technology
prototype disappeared from a secure research laboratory between 22:00 and
00:00.

The investigator can:

- View the case file
- View all suspects
- View all evidence
- Examine individual pieces of evidence
- Select a suspect
- Enter an investigation conclusion
- Submit the investigation to the API
- Receive confirmation that the investigation was stored

The full workflow runs from the browser, through the React frontend, to the
ASP.NET Core Web API, and into a SQL Server database via Entity Framework Core.
A second data access path (Dapper) is used to produce an investigation summary
that joins investigations with suspect names.

---
** 3. Technologies Used

| Layer | Technology |
|---|---|
| Backend | ASP.NET Core Web API (.NET 10) |
| Controllers | ASP.NET Core Controllers with attribute-based routing |
| ORM | Entity Framework Core |
| Alternative data access | Dapper |
| Database | SQL Server (LocalDB) |
| Frontend | React 18 + Vite |
| Frontend routing | React Router v6 |
| HTTP client | Axios |
| Styling | Custom CSS (responsive) |
| Testing | xUnit (unit + integration) |

---

** 4. API Endpoints

Base URL (development): `https://localhost:7XXX/api`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/cases` | Returns all cases |
| GET | `/api/cases/{id:int}` | Returns a single case by ID |
| GET | `/api/suspects` | Returns all suspects |
| GET | `/api/suspects/{id:int}` | Returns a single suspect by ID |
| GET | `/api/evidence` | Returns all evidence |
| GET | `/api/evidence/{id:int}` | Returns a single evidence item by ID |
| POST | `/api/investigations` | Submits a new investigation (EF Core) |
| GET | `/api/investigations/summary` | Returns investigations joined with suspect names (**Dapper**) |

**Example POST body for `/api/investigations`:**

```json
{
  "caseID": 1,
  "suspectID": 2,
  "conclusion": "Jamie Smith appears to be the most likely suspect because his access card was used to enter the laboratory shortly before the prototype disappeared."
}
```

**Example response:**

```json
{
  "message": "Investigation Submitted Successfully",
  "investigationID": 1
}
```

** Routing constraint

At least one endpoint uses a routing constraint. In this project,
`/api/cases/{id:int}`, `/api/suspects/{id:int}` and `/api/evidence/{id:int}`
all use the `:int` constraint to reject non-integer IDs with a `404`.

---

** 4. Database Setup

The database is created and seeded automatically the first time the API runs
(via `db.Database.EnsureCreated()`), but you can also create it manually using
EF Core migrations.

** Prerequisites

- SQL Server Express LocalDB (installed with Visual Studio)
- .NET SDK matching the version in the `.csproj` (`net10.0`)
- `dotnet-ef` CLI tool (install once):
  ```bash
  dotnet tool install --global dotnet-ef
  ```

** Connection string

Defined in `TracePointAPI/appsettings.json`:

```json
"ConnectionStrings": {
  "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=TracePointDB;Trusted_Connection=True;MultipleActiveResultSets=true;TrustServerCertificate=True"
}
```

** Creating the database manually (optional)

```bash
cd TracePointAPI
dotnet ef migrations add InitialCreate
dotnet ef database update
```

** Seed data

On first creation the database is seeded with:

- **1 case:** The Missing Prototype (Status: OPEN)
- **3 suspects:** Alex Morgan, Jamie Smith, Taylor Williams
- **5 evidence items:** Security Access Log, CCTV Report, Fingerprint Report,
  Email Message, Photograph

** Verifying the database

1. In Visual Studio: **View → SQL Server Object Explorer**
2. Expand `(localdb)\MSSQLLocalDB → Databases → TracePointDB → Tables`
3. Tables present: `Cases`, `Suspects`, `Evidences`, `Investigations`
4. Right-click `Cases → View Data` — you should see the seeded case.

---

** 5. Running the API

From the `TracePointAPI` folder:

```bash
cd TracePointAPI
dotnet restore
dotnet build
dotnet run
```

The terminal will print something like:

```
Now listening on: https://localhost:7123
Now listening on: http://localhost:5123
```

- API base URL: `https://localhost:7123/api`
- Swagger UI: `https://localhost:7123/swagger`


** 6. Minimum Functional Workflow (Verified)

1. Open the application (`http://localhost:5173`)
2. Click **START INVESTIGATION** → navigates to `/case`
3. View the case (loaded from `GET /api/cases/1`)
4. Click **View Suspects** → `/suspects` (loaded from `GET /api/suspects`)
5. Click **View Evidence** → `/evidence` (loaded from `GET /api/evidence`)
6. Click **Examine Evidence** on any card → modal shows full details
7. Navigate to `/investigation`
8. Select a suspect from the dropdown
9. Type a conclusion
10. Click **SUBMIT INVESTIGATION**
11. API stores the investigation via EF Core
12. Success message appears: *"Investigation Submitted Successfully"*

---

** 7. Project Structure

```
├── TracePointAPI/            # ASP.NET Core Web API (backend)
│   ├── Controllers/          # API controllers
│   ├── Data/                 # DbContext
│   ├── Migrations/           # EF Core migrations
│   ├── Models/               # Domain models
│   │   └── DTOs/             # Data Transfer Objects
│   ├── Program.cs
│   └── appsettings.json
├── TracePointClient/         # React frontend
│   ├── src/
│   │   ├── components/       # Navigation, CaseCard, SuspectCard, EvidenceCard
│   │   ├── pages/            # Home, Case, Suspects, Evidence, Investigation
│   │   ├── services/         # Axios API client
│   │   └── App.jsx
│   └── package.json
├── TracePointAPI.Tests/      # xUnit test project
└── README.md                 # This file
```


** 8. Demonstration Notes

During the demonstration, the following will be shown:

- Starting the ASP.NET Core API (`dotnet run`)
- Starting the React application (`npm run dev`)
- Displaying the case, suspects, and evidence
- Examining a piece of evidence
- Selecting a suspect and submitting an investigation
- Confirming the investigation was saved (via Swagger and SQL Server)
- Demonstrating the Dapper `summary` endpoint
- Explaining a React component (`EvidenceCard`) and its props/state
- Explaining the use of EF Core and the Dapper implementation
- Running `dotnet test` to show the 3 unit tests and 1 integration test



I declare that this assignment is my own work. AI tools were used only for
guidance and scaffolding; I fully understand and can explain all code
submitted.
