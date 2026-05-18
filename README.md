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
