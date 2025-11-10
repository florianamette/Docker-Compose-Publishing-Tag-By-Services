Docker Compose Build & Push
---
A GitHub Action that automatically builds, tags, and pushes all services from a docker-compose file to Docker Hub. **All services are published under a single Docker Hub repository**, with each service tagged using its service name (e.g., `myorg/myrepo:web`, `myorg/myrepo:api`).

## Features
- Automatically parses docker-compose files to discover all services
- Builds and pushes all services to a **single Docker Hub repository**
- Tags each service image as `<repository-name>:<service-name>` within the same repository
- Supports multiple docker-compose files (e.g., using override files for CI)
- Optional Docker Hub README update from a markdown file
- Works with GitHub organizations (uses `org/repo-name` format)

## Requirements
- Services in your docker-compose file must have a `build` property
- The `image` property is optional - the action will automatically determine image names
- Docker Hub credentials (username and token) are required

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `docker_hub_username` | Docker Hub username | Yes | - |
| `docker_hub_org_token` | Docker Hub organization/user access token or password (optional, defaults to `docker_hub_personal_token` if not provided) | No | `docker_hub_personal_token` |
| `docker_hub_org_name` | Docker Hub organization/namespace name. If not provided, defaults to `docker_hub_username` | No | `docker_hub_username` |
| `docker_hub_repository` | Docker Hub repository name. Can be full path (`org/repo-name`) or just repo name. If not provided, uses GitHub repository name | No | `${{ github.repository }}` |
| `docker_compose_files` | Docker-compose files to use (space-separated, e.g., `docker-compose.yml docker-compose.ci.yml`). The action automatically adds `-f` flags. | No | `docker-compose.yml` |
| `update_readme` | Enable updating Docker Hub repository README | No | `false` |
| `readme_path` | Path to markdown file for Docker Hub README | No | `./README.md` |
| `docker_hub_personal_token` | Docker Hub Personal Access Token (used for README updates, and as fallback for pushing images if `docker_hub_org_token` is not provided) | No | - |

## Examples

### Simple Usage

The action will automatically discover all services in your docker-compose file and publish them to Docker Hub.

**For personal accounts** (using personal token and username for both pushing and README updates):
```yaml
- name: Build and push to Docker Hub
  uses: florianamette/docker-compose-publish-auto-tagging@v1
  with:
    docker_hub_username: ${{ secrets.DOCKER_HUB_USERNAME }}
    docker_hub_personal_token: ${{ secrets.DOCKER_HUB_PERSONAL_TOKEN }}
    # docker_hub_org_token is optional - will use personal_token if not provided
    # docker_hub_org_name is optional - will use docker_hub_username if not provided
```

**For organizations** (using separate org token and org name):
```yaml
- name: Build and push to Docker Hub
  uses: florianamette/docker-compose-publish-auto-tagging@v1
  with:
    docker_hub_username: ${{ secrets.DOCKER_HUB_USERNAME }}
    docker_hub_org_name: "myorg"  # Organization name
    docker_hub_org_token: ${{ secrets.DOCKER_HUB_ORG_TOKEN }}
```

**docker-compose.yml:**
```yaml
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile.web
  api:
    build:
      context: .
      dockerfile: Dockerfile.api
  worker:
    build:
      context: .
      dockerfile: Dockerfile.worker
```

**GitHub Actions workflow:**
```yaml
name: Build and Push to Docker Hub

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build and push to Docker Hub
        uses: florianamette/docker-compose-publish-auto-tagging@v1
        with:
          docker_hub_username: ${{ secrets.DOCKER_HUB_USERNAME }}
          docker_hub_personal_token: ${{ secrets.DOCKER_HUB_PERSONAL_TOKEN }}
          # For organizations, you can use docker_hub_org_token instead
```

This will publish all images to the **same Docker Hub repository** (`your-org/your-repo`) with different tags:
- `your-org/your-repo:web`
- `your-org/your-repo:api`
- `your-org/your-repo:worker`

All services share the same repository name, with each service name used as the tag.

### Custom Docker Hub Repository Name

You can customize the Docker Hub repository name in several ways:

**Option 1: Full repository path (org/repo-name)**
```yaml
- name: Build and push to Docker Hub
  uses: florianamette/docker-compose-publish-auto-tagging@v1
  with:
    docker_hub_username: ${{ secrets.DOCKER_HUB_USERNAME }}
    docker_hub_org_token: ${{ secrets.DOCKER_HUB_ORG_TOKEN }}
    docker_hub_repository: "myorg/custom-repo-name"
```

**Option 2: Just repository name (uses docker_hub_org_name or docker_hub_username)**
```yaml
- name: Build and push to Docker Hub
  uses: florianamette/docker-compose-publish-auto-tagging@v1
  with:
    docker_hub_username: ${{ secrets.DOCKER_HUB_USERNAME }}
    docker_hub_personal_token: ${{ secrets.DOCKER_HUB_PERSONAL_TOKEN }}
    docker_hub_repository: "custom-repo-name"  # Will use docker_hub_username as org (or docker_hub_org_name if set)
```

**Option 3: Using Docker Hub organization**
```yaml
- name: Build and push to Docker Hub
  uses: florianamette/docker-compose-publish-auto-tagging@v1
  with:
    docker_hub_username: ${{ secrets.DOCKER_HUB_USERNAME }}
    docker_hub_org_token: ${{ secrets.DOCKER_HUB_ORG_TOKEN }}
    docker_hub_org_name: "myorg"  # Uses organization instead of username
    docker_hub_repository: "custom-repo-name"  # Results in: myorg/custom-repo-name
```

### Multiple Compose Files

You can use multiple docker-compose files (e.g., using override files for CI):

```yaml
- name: Build and push to Docker Hub
  uses: florianamette/docker-compose-publish-auto-tagging@v1
  with:
    docker_hub_username: ${{ secrets.DOCKER_HUB_USERNAME }}
    docker_hub_org_token: ${{ secrets.DOCKER_HUB_ORG_TOKEN }}
    docker_compose_files: 'docker-compose.yml docker-compose.ci.yml'
```


### With Docker Hub README Update

To automatically update the Docker Hub repository README from a markdown file:

```yaml
- name: Build and push to Docker Hub
  uses: florianamette/docker-compose-publish-auto-tagging@v1
  with:
    docker_hub_username: ${{ secrets.DOCKER_HUB_USERNAME }}
    docker_hub_org_token: ${{ secrets.DOCKER_HUB_ORG_TOKEN }}
    update_readme: 'true'
    docker_hub_personal_token: ${{ secrets.DOCKER_HUB_PERSONAL_TOKEN }}
    readme_path: './README.md'  # optional, defaults to ./README.md
```

**Note:** You need a Docker Hub Personal Access Token (PAT) with write permissions to update the README. The PAT is different from the regular Docker Hub token used for pushing images.

### Complete Example with README Update

```yaml
name: Build and Push to Docker Hub

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build and push to Docker Hub
        uses: florianamette/docker-compose-publish-auto-tagging@v1
        with:
          docker_hub_username: ${{ secrets.DOCKER_HUB_USERNAME }}
          docker_hub_personal_token: ${{ secrets.DOCKER_HUB_PERSONAL_TOKEN }}
          # docker_hub_org_token is optional - will use personal_token if not provided
          docker_hub_repository: "myorg/myproject"
          update_readme: 'true'
          readme_path: './docs/DOCKER_HUB_README.md'
```

## How It Works

1. **Login**: Authenticates with Docker Hub using provided credentials
2. **Parse Services**: Discovers all services from the docker-compose file(s)
3. **Build**: Builds all services using `docker compose build`
4. **Tag**: Tags each service image as `<repository-name>:<service-name>`
5. **Push**: Pushes each tagged image to Docker Hub
6. **Update README** (optional): Updates the Docker Hub repository README from a markdown file

## Image Naming

**Important:** All services are published to a **single Docker Hub repository**. The repository name is determined by the `docker_hub_repository` input (or defaults to your GitHub repository name). Each service is then tagged with its service name.

The action automatically determines the source image name for each service using multiple methods:

1. Checks `docker compose images` output (most reliable)
2. Reads the `image` property from docker-compose config
3. Tries default docker-compose naming patterns (`project-service`, `project_service`)
4. Searches all images for one matching the service name

All images are then re-tagged and pushed to the same Docker Hub repository as: `<docker-hub-repository>:<service-name>`

For example, if your repository is `myorg/myproject`, all services will be published as:
- `myorg/myproject:service1`
- `myorg/myproject:service2`
- `myorg/myproject:service3`

## Notes

- **All images are published to a single Docker Hub repository** - each service becomes a different tag in the same repository
- **Personal account fallback**: For personal accounts, you can use only `docker_hub_personal_token` and `docker_hub_username`. If `docker_hub_org_token` is not provided, the action will automatically use `docker_hub_personal_token` for pushing images. If `docker_hub_org_name` is not provided, it will use `docker_hub_username` as the namespace
- The action handles Docker Hub authentication internally - no need to login separately
- All services in the docker-compose file are processed automatically
- The action works with GitHub organizations - repository names use the `org/repo-name` format
- For README updates, you need a Docker Hub Personal Access Token with appropriate permissions
- The Docker Hub repository will be created automatically if it doesn't exist (when you push the first image)
