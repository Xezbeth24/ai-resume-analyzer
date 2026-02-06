# CI/CD Pipeline Setup Guide

## Overview

This repository includes a comprehensive CI/CD pipeline configured with GitHub Actions that automatically builds, tests, and deploys the AI Resume Analyzer application.

## Pipeline Architecture

The CI/CD pipeline consists of 4 main stages:

### 1. Code Quality Check
- **Purpose**: Ensures code quality and type safety
- **Actions**:
  - Checks out the repository code
  - Sets up Node.js 20.x environment
  - Installs dependencies using `npm ci`
  - Runs TypeScript type checking
  - Caches ESLint results for faster subsequent runs

### 2. Build Application
- **Purpose**: Creates production-ready build artifacts
- **Actions**:
  - Checks out the repository code
  - Sets up Node.js 20.x with npm caching
  - Installs dependencies
  - Builds the production bundle using `npm run build`
  - Uploads build artifacts for 7 days
  - Reports build size
- **Dependencies**: Runs after Code Quality Check passes

### 3. Docker Build and Push
- **Purpose**: Containerizes the application and pushes to Docker Hub
- **Actions**:
  - Sets up Docker Buildx for advanced builds
  - Authenticates with Docker Hub
  - Builds Docker image with layer caching
  - Pushes images with `latest` and commit SHA tags
- **Conditions**: Only runs on pushes to `main` branch
- **Dependencies**: Runs after Build Application completes

### 4. Deployment Status
- **Purpose**: Provides summary and status reporting
- **Actions**:
  - Reports status of all previous jobs
  - Creates GitHub Actions summary with pipeline results
- **Conditions**: Always runs regardless of previous job results

## Workflow Triggers

The pipeline is triggered by:

- **Push events** to `main` or `develop` branches
- **Pull requests** targeting `main` or `develop` branches
- **Manual dispatch** via GitHub Actions UI (workflow_dispatch)

## Setup Instructions

### Prerequisites

1. GitHub repository with appropriate permissions
2. Docker Hub account (optional, for Docker image push)

### Required Secrets

To enable Docker image pushing, add these secrets to your repository:

1. Go to **Settings** > **Secrets and variables** > **Actions**
2. Click **New repository secret**
3. Add the following secrets:

| Secret Name | Description | Required |
|------------|-------------|----------|
| `DOCKER_USERNAME` | Your Docker Hub username | Optional* |
| `DOCKER_PASSWORD` | Your Docker Hub access token | Optional* |

*Docker push step will be skipped if these secrets are not configured

### Creating Docker Hub Access Token

1. Log in to [Docker Hub](https://hub.docker.com/)
2. Go to **Account Settings** > **Security**
3. Click **New Access Token**
4. Give it a descriptive name (e.g., "GitHub Actions")
5. Copy the token and add it as `DOCKER_PASSWORD` secret

## Monitoring Pipeline Runs

### Viewing Workflow Runs

1. Navigate to the **Actions** tab in your repository
2. Select **CI/CD Pipeline** from the workflows list
3. View all pipeline runs with their status

### Understanding Status Indicators

- 🟡 **Yellow/Orange**: In progress
- ✅ **Green**: Success
- ❌ **Red**: Failed
- ⏸️ **Gray**: Pending/Queued

### Viewing Detailed Logs

1. Click on any workflow run
2. Click on individual job names to view logs
3. Expand steps to see detailed output

## Build Artifacts

Build artifacts are automatically generated and stored:

- **Name**: `build-artifacts`
- **Retention**: 7 days
- **Contents**: Production build output from `npm run build`

### Downloading Artifacts

1. Go to the workflow run page
2. Scroll to the **Artifacts** section at the bottom
3. Click on `build-artifacts` to download

## Docker Images

When Docker secrets are configured, images are pushed with two tags:

```
<username>/ai-resume-analyzer:latest
<username>/ai-resume-analyzer:<commit-sha>
```

### Pulling Docker Images

```bash
# Pull latest version
docker pull <username>/ai-resume-analyzer:latest

# Pull specific version
docker pull <username>/ai-resume-analyzer:<commit-sha>

# Run container
docker run -p 3000:3000 <username>/ai-resume-analyzer:latest
```

## Customization

### Modifying Node.js Version

Edit `.github/workflows/ci-cd.yml`:

```yaml
env:
  NODE_VERSION: '20.x'  # Change to desired version
```

### Adding Additional Jobs

Add new jobs in the `jobs:` section:

```yaml
my-custom-job:
  name: My Custom Job
  runs-on: ubuntu-latest
  needs: [build]  # Specify dependencies
  steps:
    - name: Custom step
      run: echo "Custom commands here"
```

### Changing Trigger Branches

Modify the `on:` section:

```yaml
on:
  push:
    branches: [ main, develop, staging ]  # Add more branches
  pull_request:
    branches: [ main ]
```

## Troubleshooting

### Common Issues

**1. TypeScript Type Check Fails**
- Check your TypeScript configuration
- Ensure all type definitions are installed
- Review error logs in the workflow run

**2. Build Fails**
- Verify `package.json` scripts are correct
- Check for missing dependencies
- Review build logs for specific errors

**3. Docker Push Fails**
- Verify Docker Hub credentials are correct
- Check that secrets are properly configured
- Ensure Docker Hub repository exists

**4. Workflow Not Triggering**
- Check branch names match trigger configuration
- Verify workflow file is in `.github/workflows/` directory
- Check workflow file syntax using YAML validator

### Getting Help

- Check [GitHub Actions documentation](https://docs.github.com/en/actions)
- Review workflow logs for error messages
- Open an issue in the repository

## Performance Optimizations

The pipeline includes several optimizations:

1. **NPM Caching**: Dependencies are cached between runs
2. **ESLint Caching**: Lint results are cached for faster checks
3. **Docker Layer Caching**: Uses GitHub Actions cache for Docker builds
4. **Parallel Jobs**: Independent jobs run in parallel when possible

## Security Best Practices

1. **Never commit secrets** to the repository
2. **Use repository secrets** for sensitive data
3. **Rotate access tokens** regularly
4. **Limit token permissions** to minimum required
5. **Review workflow changes** in pull requests carefully

## Continuous Improvement

Consider adding these features:

- [ ] Automated testing (unit, integration, e2e)
- [ ] Code coverage reporting
- [ ] Security scanning (Dependabot, Snyk)
- [ ] Performance testing
- [ ] Automated deployment to cloud platforms
- [ ] Slack/Discord notifications
- [ ] Release automation

## Additional Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Documentation](https://docs.docker.com/)
- [React Router Build Guide](https://reactrouter.com/)

---

**Last Updated**: February 2026
**Maintained By**: Repository Contributors
