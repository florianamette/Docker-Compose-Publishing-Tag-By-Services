# Docker Compose CI - Test Repository

This is a test repository for the Docker Compose CI GitHub Action.

## Test Services

This repository contains test Docker services that are automatically built and published to Docker Hub.

### Available Services

- **test-web**: Web service for testing
- **test-api**: API service for testing  
- **test-worker**: Worker service for testing

## Usage

### Pull Images

All images are published under the same Docker Hub repository with service names as tags:

```bash
# Pull web service
docker pull <your-repo>:test-web

# Pull API service
docker pull <your-repo>:test-api

# Pull worker service
docker pull <your-repo>:test-worker
```

### Run Services

```bash
# Run web service
docker run -d <your-repo>:test-web

# Run API service
docker run -d <your-repo>:test-api

# Run worker service
docker run -d <your-repo>:test-worker
```

## Testing

This README is automatically updated from the test files when the GitHub Action runs with `update_readme: true`.

**Last updated:** This README is automatically maintained by the Docker Compose CI GitHub Action.

## Repository Information

- **GitHub Action**: [docker-compose-publish-auto-tagging](https://github.com/marketplace/actions/docker-compose-publish-auto-tagging)
- **Purpose**: Test repository for CI/CD pipeline
- **Status**: Active testing

---

*This is a test repository. Images are automatically built and published via GitHub Actions.*

