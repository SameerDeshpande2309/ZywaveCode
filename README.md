**Stack**

* Backend: ASP.NET Core 7 (or .NET 6) Web API with Entity Framework Core
* Frontend: React (Vite or Create React App)
* Storage & Cloud: Azure Blob Storage, Azure Functions, Azure App Service (or Static Web Apps)
* Optional: Azure SQL (or local SQLite for local dev), Azure Key Vault (optional for production secrets)

**Project structure (suggested)**

```
/incident-mgmt
  /backend
    /IncidentApi (ASP.NET Core app)
      Controllers/
      Data/
      Models/
      Services/
      Program.cs
  /functions
    /NotificationFunction (Azure Function)
  /frontend
    /incident-frontend (React app)
  README.md
```

---

## Quick local setup (developer machine)

### Prerequisites

* .NET SDK 7 (or 6)
* Node.js 18+
* npm or yarn
* Azure CLI (for deployments)
* (Locally) SQLite or SQL Server if using local DB
* Optional: Azure Storage emulator (Azurite) for local blob storage

### Backend - local

1. Open a terminal in `/incident-mgmt/backend/IncidentApi`.
2. Create database and run migrations:

   * Using SQLite (recommended for local dev):

     * `dotnet ef migrations add InitialCreate`
     * `dotnet ef database update`
   * Or configure connection string for local SQL Server in `appsettings.Development.json`.
3. Run the API:

   * `dotnet run`
4. API swagger will be available at `https://localhost:5001/swagger` (or the console URL).

**Key endpoints**

* `POST /api/incidents` — create incident (multipart/form-data for file)
* `GET /api/incidents` — list incidents
* `PUT /api/incidents/{id}/status` — update status
* `GET /api/incidents/{id}` — get single incident (incl. attachments metadata)

### Frontend - local

1. Open `/incident-mgmt/frontend/incident-frontend`.
2. `npm install`
3. `npm run dev` (Vite) or `npm start` (CRA)
4. The app will run on `http://localhost:3000` and call the backend API.

---

## Azure configuration & deployment

### Azure services used

* **Azure App Service** — host ASP.NET Core API
* **Azure Blob Storage** — store uploaded attachments (screenshots)
* **Azure Functions** — trigger notification when a new incident is created (simple email or log)
* **Azure SQL** (optional) — production DB
* **Azure Key Vault** (optional) — store DB connection string, storage key, SendGrid key

### Steps (high level)

1. Create a Resource Group (RG).
2. Create an Azure Storage Account (Blob) and a container (e.g., `incident-attachments`).
3. Create Azure SQL or use Azure Database for MySQL/Postgres depending on preference.
4. Create App Service (Linux/Windows) and set app settings:

   * `ConnectionStrings__DefaultConnection` (from Key Vault or App settings)
   * `BlobStorage__ConnectionString` or `BlobStorage__ContainerName`
5. Deploy backend using `az webapp deploy` or GitHub Actions.
6. Create an Azure Function App and deploy `NotificationFunction`. Configure it with a binding (SendGrid) or use HTTP trigger to call SendGrid.
7. Optionally deploy frontend to **Azure Static Web Apps** or host on App Service. Configure CORS on API to allow frontend origin.

---

## Architecture / design decisions (short)

**Separation of concerns**: API handles incidents + metadata, Blob Storage holds files (cheap & scalable).
**EF Core with repository-less pattern**: small project — DbContext injected to controllers; keeps code short and readable.
Azure Function for notifications : decouples notification logic from request latency; function invoked via queue or HTTP from API.
**Secrets management**: Recommend Key Vault for production. For MVP, store secrets in App Service settings.
**Frontend**: simple React app using Axios; uploads handled as `multipart/form-data`.

---

## Notes & assumptions

  Authentication/authorization not implemented (out of scope for 3-day MVP).
  Files are public-read only if required — otherwise use SAS tokens to provide secure short-lived URLs.
  The provided Azure Function example uses SendGrid (requires SendGrid API key) — you can swap for other providers.
  Local dev uses SQLite for speed and simplicity; production should use Azure SQL for high availability.

---

## How to bundle & submit

Create repo with the structure above and push to GitHub.
Create a `zip` (zip the `incident-mgmt` folder) for submission: `zip -r incident-mgmt.zip incident-mgmt`.


