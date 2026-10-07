# TracePoint — Testing and Validation Guide

This document describes how the TracePoint system (ASP.NET Core API +
React frontend) is tested and validated. It covers the full testing
strategy: unit, integration, system, user acceptance, and regression
testing.

The goal is to ensure investigators can view cases, suspects, evidence,
and submit investigations without errors.

---

## 1. Testing Strategy Overview

TracePoint is tested at five levels:

| Level                | Scope                                                | How it is performed                              |
|----------------------|------------------------------------------------------|--------------------------------------------------|
| Unit testing         | Smallest pieces of code in isolation, no DB or net   | xUnit (API), Vitest (React)                      |
| Integration testing  | React ↔ API ↔ database                               | Vitest with mocked axios, Swagger, live app      |
| System testing       | Full end-to-end user journey                         | Manual walkthrough in browser                    |
| User testing         | Real users complete tasks unaided                    | Classmates performing scripted tasks             |
| Regression testing   | Re-running existing tests after any change           | CI + manual rerun                                |

API endpoints are validated interactively via Swagger UI. The React
frontend is validated manually in the browser and, where applicable,
with automated unit and integration tests.

---

## 2. Prerequisites

Before running any test, ensure the following are installed:

- **.NET 8 SDK** — https://dotnet.microsoft.com/download
- **Node.js LTS** (v20+) — https://nodejs.org
- **Visual Studio 2022** — for running the API
- **SQL Server Express LocalDB** — included with the ASP.NET workload

Verify the tools are on your PATH:

```
dotnet --version
node -v
npm -v
```

Each command must print a version number.

---

## 3. Running the System for Testing

Before any integration, system, or user testing, both halves of the
application must be running.

### 3.1 Start the API

1. Open `TracePointAPI.sln` in Visual Studio.
2. Select the **https** launch profile from the dropdown.
3. Press **F5** (or click the green play button).

The API will listen on:

```
https://localhost:7197
http://localhost:5187
```

Swagger UI is available at:

```
https://localhost:7197/swagger
```

### 3.2 Start the React frontend

In a separate terminal:

```
cd TracePointClient
npm install
npm run dev
```

The React app is served at:

```
http://localhost:5173/
```

### 3.3 Ensure database is created

If the API returns HTTP 500 with a message about the database not being
found, run the migration:

```
cd TracePointAPI
dotnet ef database update
```

---

## 4. Step 1 — API Endpoint Testing (Swagger)

Open Swagger at `https://localhost:7197/swagger` and validate each
endpoint using the **Try it out → Execute** button.

| Endpoint                              | Expected result                                     |
|---------------------------------------|-----------------------------------------------------|
| `GET /api/cases`                      | Returns 1 case                                      |
| `GET /api/suspects`                   | Returns 3 suspects                                  |
| `GET /api/Evidences`                  | Returns 5 evidence items                            |
| `GET /api/investigations/summary`     | Returns empty list or existing investigations       |

A successful response is HTTP **200 OK** with a JSON body.

---

## 5. Step 2 — Frontend Connection Test

1. Open `http://localhost:5173/` in a browser.
2. Confirm the **Home Page** loads with:
   - Title: **TRACEPOINT INVESTIGATIONS**
   - Subtitle: **THE MISSING PROTOTYPE**
   - Button: **START INVESTIGATION**

3. Click **START INVESTIGATION**.

**Expected:** The app navigates to the Case Page.

---

## 6. Step 3 — Case Page Validation

On the Case Page, confirm the following data loads from the API:

- **Case Name:** The Missing Prototype
- **Description:** A technology prototype has disappeared...
- **Status:** Open

Click the navigation buttons to **Suspects** and **Evidence**.
**Expected:** Navigation succeeds without errors.

---

## 7. Step 4 — Suspects Page Validation

Confirm the suspects list loads correctly:

- Alex Morgan (Software Developer)
- Jamie Smith (Security Officer)
- Taylor Williams (Research Assistant)

---

## 8. Step 5 — Evidence Page Validation

Confirm the following evidence items load:

- Security Access Log
- CCTV Report
- Fingerprint Report
- Email Message
- Photograph

---

## 9. Step 6 — Investigation Page Validation

1. Fill in the investigation form:
   - Select **Case ID**
   - Select **Suspect ID**
   - Enter a **Conclusion**
2. Click **Submit**.

**Expected:** A POST request is sent to `/api/investigations` and
returns HTTP **201 Created**.

3. Call `GET /api/investigations/summary` in Swagger.
**Expected:** The new investigation appears, including the suspect name
and conclusion.

---

## 10. Step 7 — Error Handling Tests

| Test                                                     | Expected result                              |
|----------------------------------------------------------|----------------------------------------------|
| Navigate to `/api/cases/abc`                             | HTTP **404 Not Found**                       |
| Submit investigation without required fields             | HTTP **400 Bad Request**                     |
| Stop the API and reload the React app                    | App displays a connection error message      |

The frontend shows a message such as:

> "Failed to load case data. Please ensure the API is running."

---

## 11. Step 8 — Final Validation

- Confirm the **CORS policy** allows the React frontend
  (`http://localhost:5173`) to call the API
  (`https://localhost:7197`).
- Verify database tables contain seeded data and any newly submitted
  investigations.
- Confirm there are **no console errors** in the React app (open F12
  in the browser and check the Console tab).
- Confirm all Swagger endpoints respond successfully.

---

## 12. Step 9 — Unit Testing

Unit tests verify the smallest pieces of code **in isolation**, with no
database and no network.

### 12.1 API unit tests (xUnit)

- Test that the `CasesController` action returns the seeded case when
  given a mock or in-memory repository.
- Each test must pass on its own.

Run from the solution root:

```
dotnet test
```

### 12.2 React unit tests (Vitest + React Testing Library)

- Test that the Suspects page displays **Alex Morgan**, **Jamie Smith**,
  and **Taylor Williams** when given sample data.
- Each test must pass on its own.

Install the testing tools (once):

```
cd TracePointClient
npm install -D vitest @testing-library/react @testing-library/jest-dom @testing-library/user-event jsdom
```

Run the tests:

```
npm test
```

Expected output:

```
 ✓ src/pages/Suspects.test.jsx
 Test Files  1 passed (1)
      Tests  1 passed (1)
```

---

## 13. Step 10 — Integration Testing

Integration tests verify that separate parts work together.

| Integration                          | How it is verified                                              |
|--------------------------------------|-----------------------------------------------------------------|
| React → API (suspects)               | Suspects page loads from `/api/suspects`                        |
| React → API (evidence)               | Evidence page loads from `/api/Evidences`                       |
| API → database (write)               | Submitting an investigation saves a record                      |
| API → database (read)                | The summary endpoint returns the newly submitted investigation  |
| CORS                                 | React calls succeed with no CORS error in the browser console   |

Each integration scenario is performed with both applications running.

---

## 14. Step 11 — System Testing (End-to-End)

Test the complete application in the browser, the way a real user
would.

Run both the API and the React app, then follow this flow:

1. Home page loads
2. Click **START INVESTIGATION**
3. Case page loads
4. Navigate to **Suspects**
5. Navigate to **Evidence**
6. Navigate to **Investigation**
7. Submit an investigation
8. Open the summary endpoint and confirm the new investigation appears

**Expected:** The whole flow completes with no errors.

---

## 15. Step 12 — User Testing

Ask **2–3 classmates** to use the system without any help.

Give them three tasks:

1. Find the case details.
2. List the suspects.
3. Submit an investigation.

Observe where they hesitate or become confused (unclear buttons,
confusing form fields) and record their feedback. Fix any issues they
identify, then re-run the relevant regression tests.

---

## 16. Step 13 — Regression Testing

After **any** fix or change, re-run the existing tests to make sure
nothing that previously worked is now broken.

1. Re-run all unit tests:
   - `dotnet test`
   - `npm test`
2. Repeat the quick checks from Steps 1–8:
   - Swagger endpoints
   - Page navigation
   - Error handling

**Example:** After fixing the investigation form, confirm the Suspects
and Evidence pages still load.

---

## 17. Summary of Testing Coverage

The following testing activities have been completed:

- Tested API endpoints with Swagger
- Validated React frontend navigation and data loading
- Confirmed case, suspects, and evidence display correctly
- Verified investigation submission and summary retrieval
- Checked error handling and routing constraints
- Ensured database and CORS configuration work properly
- Applied unit, integration, system, user, and regression testing

---

## 18. Troubleshooting

| Problem                                        | Solution                                                                |
|------------------------------------------------|-------------------------------------------------------------------------|
| Swagger returns HTTP 500 on every endpoint     | Run `dotnet ef database update` in the `TracePointAPI` folder           |
| Build fails with "file in use"                 | Stop the running backend in Visual Studio (Shift+F5)                    |
| React shows "Failed to load case data"         | Confirm the API is running on `https://localhost:7197`                  |
| CORS error in browser console                  | Ensure the client is running on `http://localhost:5173`                 |
| `vitest` not found                             | Run `npm install` inside `TracePointClient`                             |
| `dotnet ef` not found                          | Install with `dotnet tool install --global dotnet-ef`                   |
| SSL certificate warning                        | Run `dotnet dev-certs https --trust` once and restart the API           |

---

## 19. Submission Evidence

For the final submission, provide:

- A screenshot of Swagger showing `GET /api/cases` returning 200 OK.
- A screenshot of the Home Page in the browser.
- A screenshot of the Suspects and Evidence pages.
- A screenshot of a submitted investigation appearing in the summary.
- A screenshot of `npm test` (or `dotnet test`) showing all tests
  passing.
