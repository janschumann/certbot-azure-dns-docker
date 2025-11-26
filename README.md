# certbot-azure-dns-docker

A Docker image for Certbot with Azure DNS plugin support. This image extends the official Certbot image and includes the `certbot-dns-azure` plugin and its dependencies.

## Docker Hub Image

The image is available on Docker Hub as:
```
janschumann/certbot-azure-dns
```

## What's Included

This project builds multiple image variants:

**Latest variant** (unpinned):
- Base image: `certbot/certbot:latest`
- `certbot-dns-azure` (latest version)
- `azure-mgmt-dns` (latest version)

**Versioned variants** (pinned):
- Base image: `certbot/certbot:v5.1.0`
- `certbot-dns-azure==2.6.1` - Azure DNS plugin for Certbot
- `azure-mgmt-dns==8.2.0` - Azure DNS management library

Additional versioned variants are added as new Certbot versions or plugin updates are released.

## Usage

### Pull the image

You can pull different tag variants depending on your needs:

```bash
# Latest variant (unpinned, tracks latest versions)
docker pull janschumann/certbot-azure-dns:latest

# Specific Certbot version (latest build for that version)
docker pull janschumann/certbot-azure-dns:5.1.0

# Certbot version + Azure plugin version
docker pull janschumann/certbot-azure-dns:5.1.0-azure2.6.1

# Full specification (Certbot + all plugin versions)
docker pull janschumann/certbot-azure-dns:5.1.0-azure2.6.1-azmgmt8.2.0

# Major/minor version aliases (point to latest in that range)
docker pull janschumann/certbot-azure-dns:5.1
docker pull janschumann/certbot-azure-dns:5
```

**Tag format**: `<certbot-version>[-azure<plugin-version>][-azmgmt<mgmt-version>]`

### Run Certbot with Azure DNS

Example usage with Azure credential file:

```bash
docker run -it --rm --name certbot \
  -v "/path/to/azure.ini:/root/azure.ini" \
  -v "/path/to/etc/letsencrypt:/etc/letsencrypt" \
  -v "/path/to/var/lib/letsencrypt:/var/lib/letsencrypt" \
  -v "/path/to/var/log:/var/log" \
  janschumann/certbot-azure-dns certonly \
    --authenticator dns-azure \
    --preferred-challenges dns \
    --noninteractive \
    --agree-tos \
    --dns-azure-config /root/azure.ini \
    --register-unsafely-without-email \
    --keep-until-expiring \
    --domains foo.example.com
```

The Azure credential file (`azure.ini`) should look like:

**Single zone configuration:**
```ini
dns_azure_client_id = your-client-id
dns_azure_client_secret = your-client-secret
dns_azure_tenant_id = your-tenant-id
dns_azure_resource_group = your-resource-group
```

**Multiple zones configuration:**
```ini
dns_azure_sp_client_id = '<client id>'
dns_azure_sp_client_secret = '<client secret>'
dns_azure_tenant_id = '<tenant id>'
dns_azure_environment = "AzurePublicCloud"

dns_azure_zone1 = foo.example.com:/subscriptions/<subscription-id>/resourceGroups/<resource-group>
dns_azure_zone2 = example.com:/subscriptions/<subscription-id>/resourceGroups/<resource-group>
```

Replace `<subscription-id>` and `<resource-group>` with your Azure subscription ID and resource group name respectively.

## Building Locally

To build the image locally, you can use build arguments to specify versions:

```bash
# Build latest variant (unpinned)
docker build -t janschumann/certbot-azure-dns:latest \
  --build-arg CERTBOT_VERSION=latest \
  --build-arg CERTBOT_DNS_AZURE_VERSION= \
  --build-arg AZURE_MGMT_DNS_VERSION= .

# Build versioned variant
docker build -t janschumann/certbot-azure-dns:5.1.0 \
  --build-arg CERTBOT_VERSION=v5.1.0 \
  --build-arg CERTBOT_DNS_AZURE_VERSION=2.6.1 \
  --build-arg AZURE_MGMT_DNS_VERSION=8.2.0 .
```

## Versioning and Tags

This project uses a versioning scheme aligned with the upstream Certbot version:

- **Format**: `<certbot-version>[-azure<plugin-version>][-azmgmt<mgmt-version>]`
- **Examples**: 
  - `5.1.0` - Latest build for Certbot 5.1.0
  - `5.1.0-azure2.6.1` - Certbot 5.1.0 with certbot-dns-azure 2.6.1
  - `5.1.0-azure2.6.1-azmgmt8.2.0` - Full version specification
  - `5.1` - Latest 5.1.x build
  - `5` - Latest 5.x.x build
  - `latest` - Unpinned variant (uses latest Certbot and plugins)

When Certbot or plugins are updated, new version combinations are added to the build matrix.

## CI/CD

This project uses GitHub Actions to automatically build and push Docker images:

**Build behavior:**
- All pushes to `main` branch trigger builds for all matrix combinations (to verify everything works)
- Pull requests trigger builds for all matrix combinations (no push)
- Builds verify that all version combinations compile successfully

**Push behavior:**
- Only tag builds (tags starting with `v*`) trigger pushes to Docker Hub
- Regular pushes to `main` build but do not push (ensures only tagged releases are published)

**Available tags per build:**
Each versioned build creates multiple tags:
- Exact version: `5.1.0`
- With plugin version: `5.1.0-azure2.6.1`
- Full specification: `5.1.0-azure2.6.1-azmgmt8.2.0`
- Version aliases: `5.1`, `5` (point to latest in that range)

### Setting up GitHub Secrets

To enable Docker Hub pushes, you need to configure GitHub secrets with a Docker Hub Personal Access Token (PAT):

1. **Create a Docker Hub Personal Access Token:**
   - Go to [Docker Hub Account Settings](https://hub.docker.com/settings/security)
   - Click "New Access Token"
   - Give it a name (e.g., "github-actions")
   - Set permissions to "Read, Write, Delete" (or at least "Read, Write")
   - Copy the token (you won't be able to see it again)

2. **Add GitHub Secrets:**
   - Go to your GitHub repository → Settings → Secrets and variables → Actions
   - Add the following secrets:
     - `DOCKER_USERNAME`: Your Docker Hub username
     - `DOCKER_PASSWORD`: Your Docker Hub Personal Access Token (not your password)

**Note:** Docker Hub requires Personal Access Tokens (PAT) for API access. Regular passwords will not work.

## Requirements

To use the Azure DNS plugin, you need:
- An Azure subscription
- Azure Active Directory application registration with DNS Zone Contributor role
- Client ID, Client Secret, and Tenant ID from the Azure AD app registration

## License

This project extends the Certbot Docker image and follows the same licensing as the upstream project.
