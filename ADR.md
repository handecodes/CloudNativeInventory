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
- Scales to zero which I pay nothing when the API isn't being used
- Managed Identity is straightforward to set up, which I need for Key Vault
- No need to manage servers or Kubernetes myself

## Security decisions
I run the container as non-root (USER app in Dockerfile) to reduce the
attack surface. If someone were to break into the container, they can't
affect the rest of the system as easily. Secrets like API keys are not
stored in code but fetched from Azure Key Vault via Managed Identity
at startup.

## Consequences
This choice means I need to set up Azure Container Registry to store
my Docker images, and connect ACA to Key Vault via a Managed Identity
with minimal permissions (Key Vault Secrets User role).