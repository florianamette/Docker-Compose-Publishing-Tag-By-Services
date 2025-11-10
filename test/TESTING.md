# Testing the Docker Compose Action

This guide explains how to test the GitHub Action locally and in GitHub Actions.

## Prerequisites

1. **Docker Hub credentials** - You need:
   - `DOCKER_HUB_USERNAME` - Your Docker Hub username
   - `DOCKER_HUB_ORG_TOKEN` - Docker Hub organization/user access token (for pushing images, optional if using personal token)
   - `DOCKER_HUB_ORG_NAME` - Docker Hub organization/namespace name (optional, defaults to username)
   - `DOCKER_HUB_PERSONAL_TOKEN` - Docker Hub Personal Access Token (for README updates, and as fallback for pushing images)
   - `DOCKER_HUB_TEST_REPOSITORY` (optional) - A test repository name (e.g., `your-org/test-repo`)

2. **Set up GitHub Secrets**:
   - Go to your repository Settings → Secrets and variables → Actions
   - Add the secrets listed above

## Test Workflows

### 1. Basic Test (`test.yml`)

Tests the basic functionality with both organization and personal account examples:
- Automatically triggers on push to `main` or `test` branches
- Can be manually triggered via workflow_dispatch
- Includes two test steps:
  - **Organization example**: Uses `docker_hub_org_token` and `docker_hub_org_name`
  - **Personal account example**: Uses only `docker_hub_personal_token` (demonstrates fallback behavior)

**To run:**
- Push to `main` or `test` branch, OR
- Go to Actions tab → "Test Docker Compose Action" → Run workflow

**Note**: The personal account test demonstrates that `docker_hub_org_token` and `docker_hub_org_name` are optional and will fallback to `docker_hub_personal_token` and `docker_hub_username` respectively.

### 2. Test README Update (`test-readme-update.yml`)

Tests the README update functionality:
- Manual trigger only (for safety)
- Updates Docker Hub repository README from `test/README.md`

**To run:**
- Go to Actions tab → "Test README Update" → Run workflow
- Make sure `DOCKER_HUB_PERSONAL_TOKEN` secret is set
- The workflow will update the Docker Hub repository README with content from `test/README.md`

### 3. Test Personal Account (`test-personal-account.yml`)

Tests the personal account fallback functionality:
- Manual trigger only (workflow_dispatch)
- Demonstrates using only `docker_hub_personal_token` without `docker_hub_org_token` or `docker_hub_org_name`
- Shows that the action automatically falls back to personal token and username

**To run:**
- Go to Actions tab → "Test Personal Account (Fallback)" → Run workflow
- Make sure `DOCKER_HUB_PERSONAL_TOKEN` secret is set

### 4. Test Organization (`test-organization.yml`)

Tests the organization functionality with explicit `docker_hub_org_name`:
- Manual trigger only (workflow_dispatch)
- Demonstrates using `docker_hub_org_name` to publish to a different organization/namespace than the username
- Shows how to use organization credentials explicitly

**To run:**
- Go to Actions tab → "Test Organization (with org_name)" → Run workflow
- Make sure `DOCKER_HUB_ORG_NAME` and `DOCKER_HUB_ORG_TOKEN` secrets are set
- This test shows how images are published to `{org_name}/{repository}` instead of `{username}/{repository}`

## Local Testing (Alternative)

You can also test the action logic locally using `act` (GitHub Actions local runner):

```bash
# Install act (if not already installed)
# macOS: brew install act
# Linux: See https://github.com/nektos/act#installation

# Test the action locally
act -s DOCKER_HUB_USERNAME=your-username \
    -s DOCKER_HUB_ORG_TOKEN=your-token \
    -W .github/workflows/test.yml
```

## Test Files

All test files are located in the `test/` folder:
- `test/docker-compose.test.yml` - Test compose file with 3 services
- `test/Dockerfile.test` - Simple test Dockerfile
- `test/README.md` - Test README file for testing Docker Hub README update functionality

## What to Check

After running tests, verify:

1. **Images are pushed to Docker Hub:**
   - Check your Docker Hub repository
   - Verify images are tagged correctly: `<repo>:test-web`, `<repo>:test-api`, etc.

2. **Services are discovered correctly:**
   - Check workflow logs for "Found services: ..."
   - Verify all expected services are listed

3. **README update (if tested):**
   - Check Docker Hub repository README is updated

## Troubleshooting

- **Authentication errors:** Verify Docker Hub credentials are correct
- **Service not found:** Check docker-compose file syntax
- **Image not found:** Verify build step completed successfully
- **README update fails:** Check PAT has write permissions

