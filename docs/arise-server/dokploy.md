# DokPloy

DokPloy is the container orchestration UI running on arise-server.

## Access

- **URL**: https://dokploy.arisegroup-tools.com
- **Login**: Use your DokPloy credentials

## Key Concepts

| Term | Description |
|------|-------------|
| Project | Group of related services (e.g., "n8n-sandboxes") |
| Environment | Subdivision within a project (usually "production") |
| Compose | Docker Compose service definition |
| Domain | Hostname routing to a service |

## Creating a New Service

1. **Create Project** (if needed)
   - Settings → Projects → Create Project
   - Name it descriptively (e.g., "client-demos")

2. **Create Compose**
   - Within project → Add Compose
   - Paste docker-compose.yml content
   - Set environment variables

3. **Deploy**
   - Click Deploy button
   - Wait for containers to start

4. **Add Domain** (for public access)
   - Go to service → Domains → Add Domain
   - See [Deploying Services](deploying-services.md) for details

## User Management

### Known Bug: Admin Role Doesn't Grant Permissions

DokPloy has a bug where promoting a user to "Admin" via the UI does NOT enable their granular permissions.

**Symptoms:**
- User is shown as "Admin" in Settings → Users
- User cannot create new projects (buttons missing)

**Fix:** Use the API to set permissions:

```bash
# Get user ID
curl -s -H "x-api-key: $DOKPLOY_API_KEY" "$DOKPLOY_URL/user.all" | jq

# Set permissions
curl -s -X POST -H "x-api-key: $DOKPLOY_API_KEY" -H "Content-Type: application/json" \
  "$DOKPLOY_URL/user.assignPermissions" -d '{
  "id": "<user-id>",
  "canCreateProjects": true,
  "canCreateServices": true,
  "canDeleteProjects": true,
  "canDeleteServices": true,
  "canAccessToDocker": true,
  "canAccessToAPI": true,
  "canAccessToSSHKeys": true,
  "canAccessToGitProviders": true,
  "canAccessToTraefikFiles": true,
  "canCreateEnvironments": true,
  "canDeleteEnvironments": true,
  "accessedProjects": [],
  "accessedEnvironments": [],
  "accessedServices": []
}'
```

Have the user log out and back in after.

## API Usage

DokPloy has a REST API for automation.

### Authentication
```bash
export DOKPLOY_URL="https://dokploy.arisegroup-tools.com/api"
export DOKPLOY_API_KEY="your-api-key"
```

### Common Operations

```bash
# List all projects
curl -s -H "x-api-key: $DOKPLOY_API_KEY" "$DOKPLOY_URL/project.all" | jq

# Deploy a compose
curl -s -X POST -H "x-api-key: $DOKPLOY_API_KEY" -H "Content-Type: application/json" \
  "$DOKPLOY_URL/compose.deploy" -d '{"composeId":"<id>"}'

# Add domain to compose
curl -s -X POST -H "x-api-key: $DOKPLOY_API_KEY" -H "Content-Type: application/json" \
  "$DOKPLOY_URL/domain.create" -d '{
  "composeId": "<compose-id>",
  "host": "myservice.arisegroup-tools.com",
  "port": 5678,
  "serviceName": "n8n",
  "domainType": "compose",
  "https": false,
  "path": "/"
}'
```

## Related Docs

- [Networking](networking.md) - How Traefik routing works
- [Deploying Services](deploying-services.md) - Step-by-step guide
