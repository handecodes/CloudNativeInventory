# ADR-001: Choosing an Azure Service for Container Hosting

## Status
Accepted

## Context
I need to deploy my containerized .NET 9 API in Azure.
I compared two options: Azure Container Apps and Azure App Service.

## Decision
I chose **Azure Container Apps**.

## Why not Azure App Service?
App Service works well for traditional web apps but isn't really built
for containers from the ground up. It feels like forcing a container
into something that wasn't designed for it. It also costs money even
when nobody is using the app. And I don't have money, I'm a student.

## Why Azure Container Apps?
- Built specifically for containers, which fits my project better
- Scales to zero which means I pay nothing when the API isn't being used
- Managed Identity is straightforward to set up, which I need for Key Vault
- No need to manage servers or Kubernetes myself

## Consequences
This choice means I need to set up Azure Container Registry to store
my Docker images, and connect ACA to Key Vault via a Managed Identity
with minimal permissions (Key Vault Secrets User role).

## Security implementation

### Rootless container
The container runs as a non-root user (USER app in Dockerfile). This means
that if someone were to break into the running container, they would not have
root/admin access to the underlying system. This limits what damage can be
done and reduces the overall attack surface.

### Secret management
Secrets are never stored in code or version-controlled files. The VendorApiKey
is stored in Azure Key Vault and fetched at runtime via the configuration
provider. The KeyVaultUrl is the only value in appsettings.json, and it is
not sensitive.

### Managed Identity and least privilege (RBAC)
The container app uses a System-Assigned Managed Identity to authenticate
to Azure Key Vault without any passwords or credentials. The identity was
given only the Key Vault Secrets User role, which allows it to read secrets
but nothing else — it cannot create, delete or manage the vault. This follows
the principle of least privilege: the app gets exactly the permissions it
needs, and nothing more.

### Verification
After deployment, GET /api/inventory/system/verify-integration returns
HTTP 200 OK with "Status": "Secured", confirming that the secret was
successfully loaded from Key Vault and not from local configuration.
