# CloudNativeInventory

A production-ready .NET 9 Inventory API containerized with Docker and deployed
to Azure using CI/CD, secure secret management, and cloud-native principles.

## Azure services used
- **Azure Container Registry** — stores Docker images
- **Azure Container Apps** — hosts the containerized API
- **Azure Key Vault** — stores secrets securely
- **Azure Managed Identity** — authenticates the app to Key Vault without passwords
- **Log Analytics Workspace** — monitoring and logs

---

## Run locally

1. Clone the repo
2. Navigate to the API project:
   ```
   cd src/CloudNativeInventory.Api
   ```
3. Run the API:
   ```
   dotnet run
   ```
4. Test the endpoint:
   ```
   curl http://localhost:5000/api/inventory
   ```

> No secrets are needed to run locally. The app uses an in-memory database
> and skips Key Vault if KeyVaultUrl is not set.

### Run with Docker locally
```
docker build -t cloudnativeinventory -f src/CloudNativeInventory.Api/Dockerfile src/CloudNativeInventory.Api
docker run -p 8080:8080 cloudnativeinventory
curl http://localhost:8080/api/inventory
```

---

## CI/CD pipeline

Two workflows in `.github/workflows/`:

### ci.yml — Continuous Integration
- **Triggers:** push or pull request to `main`
- **Steps:**
  1. Checkout code
  2. Setup .NET 9
  3. dotnet restore
  4. dotnet build
  5. dotnet test

The pipeline blocks merging if any test fails (enforced via branch ruleset).

### deploy.yml — Continuous Deployment
- **Triggers:** push to `main` only
- **Steps:**
  1. Login to Azure Container Registry
  2. Build and push Docker image (tagged with commit SHA)
  3. Login to Azure
  4. Deploy image to Azure Container Apps

---

## Deployment and verification

After every merge to `main` the deploy pipeline runs automatically.

To verify the deployment is working and secrets are loaded securely:

```
GET https://ca-inventory-api.icysea-5b3a24a1.germanywestcentral.azurecontainerapps.io/api/inventory/system/verify-integration
```

Expected response:
```json
{"status": "Secured", "message": "Hemlighet laddades framgångsrikt via säker konfiguration."}
```

If it returns `"Status": "Unsecured"` the Key Vault integration is not working.

To verify the inventory endpoint:
```
GET https://ca-inventory-api.icysea-5b3a24a1.germanywestcentral.azurecontainerapps.io/api/inventory
```

---

## Architecture Decision Record

See [ADR.md](./ADR.md) for full motivation of infrastructure and security decisions including:
- Choice of Azure Container Apps over Azure App Service
- Rootless container configuration
- Secret management via Key Vault
- Managed Identity and least privilege (RBAC)
