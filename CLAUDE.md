# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Helm chart repository for deploying Supabase on Kubernetes. Supabase is an open-source Firebase alternative that provides a complete backend platform with PostgreSQL database, authentication, realtime subscriptions, storage, and serverless functions.

## Architecture

The chart deploys multiple interconnected services:
- **Kong** (API Gateway) - Routes external traffic to internal services
- **PostgreSQL** - Core database with Supabase extensions
- **Auth** - Handles authentication flows and user management
- **PostgREST** - Automatically generates REST APIs from PostgreSQL schemas
- **Realtime** - Provides WebSocket-based subscriptions to database changes
- **Storage** - S3-compatible file storage API
- **Studio** - Web-based admin dashboard
- **Analytics/Vector** - Log aggregation and analytics

All services communicate internally via Kubernetes services and are configured through a combination of ConfigMaps, Secrets, and environment variables.

## Key Commands

### Development & Testing

```bash
# Lint the Helm chart
docker run -it \
  --workdir=/data \
  --volume $(pwd)/charts/supabase:/data \
  quay.io/helmpack/chart-testing:v3.7.1 \
  ct lint --validate-maintainers=false --chart-dirs . --charts .

# Test chart installation
docker run -it \
  --network host \
  --workdir=/data \
  --volume ~/.kube/config:/root/.kube/config:ro \
  --volume $(pwd)/charts/supabase:/data \
  quay.io/helmpack/chart-testing:v3.7.1 \
  ct install --chart-dirs . --charts .

# Package the chart and update repository index
./build.sh
```

### Deployment

```bash
# Install the chart with example values
helm install demo -f charts/supabase/values.example.yaml charts/supabase

# Upgrade an existing deployment
helm upgrade <release-name> -f <values-file> charts/supabase

# Check deployment status
kubectl get pod -l app.kubernetes.io/instance=<release-name>
```

## Important Implementation Details

1. **Template Structure**: Each service has its own template directory under `charts/supabase/templates/`. When modifying a service, ensure you update all related resources (Deployment/StatefulSet, Service, ConfigMap, etc.)

2. **Secret Management**: The chart uses a centralized secret template (`templates/secrets/secret.yaml`) that aggregates all sensitive configurations. JWT secrets and database passwords must be properly configured for security.

3. **Service Dependencies**: Services have specific startup dependencies. The database must be ready before other services start. This is handled through init containers that wait for service availability.

4. **Configuration Hierarchy**: Values flow from `values.yaml` → service-specific configurations → environment variables. Always check the default `values.yaml` to understand available options.

5. **Database Migrations**: The chart includes automatic database migration handling through init containers. Custom migrations can be added via ConfigMaps.

## Common Development Tasks

When modifying the chart:
- Update `Chart.yaml` version when making changes
- Test changes using the linting and installation commands above
- Ensure all template changes are valid YAML by running `helm template`
- Check generated manifests with `helm install --dry-run --debug`

When adding new features:
- Follow the existing pattern of organizing templates by service
- Use consistent naming: `<service>-deployment.yaml`, `<service>-service.yaml`, etc.
- Leverage existing helper templates in `_helpers.tpl` for common patterns
- Document new values in `values.yaml` with clear descriptions