# TracePoint-Investigation-API - Digital Investigation System

# 1. Project Overview

# 1.1 Introduction

**TracePoint Investigations** is a fictional private investigation company
that requires a bespoke digital investigation system. The system was
commissioned to replace a manual, paper-based case-file workflow with a
browser-accessible, database-driven platform that allows a single investigator
to open a case, review the evidence, examine suspects, and formally submit a
conclusion.

This project delivers that system end-to-end as a **full-stack web
application**, built in accordance with the NWED622 Practical Assignment
specification. It demonstrates competence across the full modern web
development stack: REST API design, relational database modelling, ORM
integration, alternative raw-SQL data access, component-based frontend
development, client-side routing, state management, and automated testing.

# 1.2 Business Problem

Before this system, TracePoint Investigations had no consistent way to:

- Store case data in one central location
- Link suspects and evidence to a specific case
- Record which suspect an investigator had concluded was most likely
- Keep an audit trail of when each investigation was submitted
- Allow an investigator to work from any device connected to the company
  network
- Retrieve a combined view of all investigations alongside the name of the
  suspect each one points to

The absence of a system meant that case files could be lost, evidence could
be misfiled, and conclusions could not be traced back to a timestamp or a
particular investigator. This project solves each of those problems through a
structured, relational database and a browser-based interface.

# 1.3 Scope of the System

The application implements the following functional capabilities:

| # | Capability | Where implemented |
|---|---|---|
| 1 | View the case file | `/case` — `CasePage.jsx` calls `GET /api/cases/1` |
| 2 | View all suspects | `/suspects` — `SuspectsPage.jsx` calls `GET /api/suspects` |
| 3 | View all evidence | `/evidence` — `EvidencePage.jsx` calls `GET /api/evidence` |
| 4 | Examine an individual piece of evidence | Modal in `EvidencePage.jsx`, driven by `useState` |
| 5 | Select a suspect | Dropdown in `InvestigationPage.jsx`, driven by `useState` |
| 6 | Enter an investigation conclusion | Textarea in `InvestigationPage.jsx`, driven by `useState` |
| 7 | Submit the investigation | Form POST to `/api/investigations` |
| 8 | Store the investigation in the database | `InvestigationsController` → EF Core → SQL Server |
| 9 | Display a confirmation message | Success alert in `InvestigationPage.jsx` |
| 10 | View investigations joined with suspect names | `GET /api/investigations/summary` (Dapper) |

The scope deliberately excludes user authentication, role-based access,
multiple concurrent cases, and evidence upload — none of which are required
by the assignment brief. The design nonetheless leaves clear extension points
for those features.

# 2. Application Description

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
# 3. Technologies Used

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

# 4. Project Structure

```text
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
└── README.md
```

# 5. Main Folders

# Backend — `TracePointAPI/`

| Folder | Responsibility |
|---|---|
| `Controllers/` | One controller per resource. Each exposes attribute-routed HTTP endpoints returning JSON. |
| `Data/` | Contains `TracePointContext` (EF Core `DbContext`). Responsible for connecting to SQL Server, defining `DbSet`s, and seeding initial case data. |
| `Migrations/` | Auto-generated by `dotnet ef migrations add`. Contains the code that creates tables and inserts seed rows. |
| `Models/` | Domain entities (`Case`, `Suspect`, `Evidence`, `Investigation`) mapping directly to database tables. |
| `Models/DTOs/` | Data Transfer Objects shaping API input (`CreateInvestigationDto`) and Dapper output (`InvestigationSummaryDto`). |

# Frontend — `TracePointClient/src/`

| Folder | Responsibility |
|---|---|
| `components/` | Reusable presentational components (`Navigation`, `CaseCard`, `SuspectCard`, `EvidenceCard`). Each receives data via props. |
| `pages/` | One file per route. Pages fetch data from the API, hold state, and compose components. |
| `services/` | `api.js` centralises all Axios calls. Changing the API base URL is a one-line edit. |

# Tests — `TracePointAPI.Tests/`

| File | Responsibility |
|---|---|
| `Validation.cs` | Extracted pure validation logic. |
| `UnitTests.cs` | 3 unit tests verifying submit-form validation rules. |
| `IntegrationTests.cs` | 1 integration test posting a real investigation to the in-memory API. |

# 6. Database Design

# Database engine

SQL Server Express **LocalDB** — created and seeded automatically on first
API startup via `db.Database.EnsureCreated()`.

# Tables

| Table | Purpose | Primary Key |
|---|---|---|
| `Cases` | Investigation case files | `CaseID` (int, identity) |
| `Suspects` | Persons of interest | `SuspectID` (int, identity) |
| `Evidences` | Collected evidence items | `EvidenceID` (int, identity) |
| `Investigations` | Submitted investigation records | `InvestigationID` (int, identity) |

# Entity relationships

```
    Cases                          Suspects
  ┌─────────┐                    ┌──────────────┐
  │ CaseID  │◄──────┐     ┌─────►│  SuspectID   │
  │ CaseName│       │     │      │  Name        │
  │ Desc    │       │     │      │  Occupation  │
  │ Status  │       │     │      │  Description │
  └─────────┘       │     │      └──────────────┘
                    │     │
              ┌─────┴─────┴──────┐
              │  Investigations  │
              │  InvestigationID │
              │  CaseID          │ ← FK → Cases.CaseID
              │  SuspectID       │ ← FK → Suspects.SuspectID
              │  Conclusion      │
              │  DateStarted     │
              └──────────────────┘

           Evidences  (standalone)
           ┌──────────────────┐
           │  EvidenceID      │
           │  Title           │
           │  Description     │
           │  Location        │
           └──────────────────┘
```

- **One Case → Many Investigations**
- **One Suspect → Many Investigations**
- **Evidence** is standalone in this assignment

# Seed data

| Table | Rows | Content |
|---|---|---|
| `Cases` | 1 | The Missing Prototype (OPEN) |
| `Suspects` | 3 | Alex Morgan, Jamie Smith, Taylor Williams |
| `Evidences` | 5 | Access log, CCTV, fingerprint, email, photo |
| `Investigations` | 0 | Populated as the investigator submits |

# 7. Database Entities

# 7.1 `Case`

| Property | Type | Description |
|---|---|---|
| `CaseID` | int | Primary key, identity |
| `CaseName` | string | Name of the case |
| `Description` | string | Full case description |
| `Status` | string | OPEN / CLOSED (defaults to OPEN) |

# 7.2 `Suspect`

| Property | Type | Description |
|---|---|---|
| `SuspectID` | int | Primary key, identity |
| `Name` | string | Full name |
| `Occupation` | string | Job title |
| `Description` | string | Background / context |

# 7.3 `Evidence`

| Property | Type | Description |
|---|---|---|
| `EvidenceID` | int | Primary key, identity |
| `Title` | string | Short title |
| `Description` | string | Full details |
| `Location` | string | Where found / stored |

# 7.4 `Investigation`

| Property | Type | Description |
|---|---|---|
| `InvestigationID` | int | Primary key, identity |
| `CaseID` | int | Foreign key → `Cases.CaseID` |
| `SuspectID` | int | Foreign key → `Suspects.SuspectID` |
| `Conclusion` | string | Investigator's conclusion |
| `DateStarted` | DateTime | Submission timestamp |
| `Case` | Case? | Navigation property |
| `Suspect` | Suspect? | Navigation property |

# 7.5 DTOs

**`CreateInvestigationDto`** — POST input shape:

| Property | Type |
|---|---|
| `CaseID` | int |
| `SuspectID` | int |
| `Conclusion` | string |

**`InvestigationSummaryDto`** — Dapper output shape:

| Property | Type |
|---|---|
| `InvestigationID` | int |
| `SuspectName` | string |
| `Conclusion` | string |
| `DateStarted` | DateTime |

# 8. API Endpoints

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

# Example POST body for `/api/investigations`:**

```json
{
  "caseID": 1,
  "suspectID": 2,
  "conclusion": "Jamie Smith appears to be the most likely suspect because his access card was used to enter the laboratory shortly before the prototype disappeared."
}
```

I declare that this assignment is my own work. AI tools were used only for
guidance and scaffolding; I fully understand and can explain all code
submitted.
