# GitHub Actions Workflows

This directory contains GitHub Actions workflows for automated CI/CD.

## Deploy to Azure Container Registry

The `deploy-to-acr.yml` workflow automatically builds and deploys the application to Azure Container Registry when changes are pushed to the main branch.

### Prerequisites

Before the workflow can run successfully, you need to configure the following GitHub Secrets in your repository:

1. **ACR_LOGIN_SERVER**: Your Azure Container Registry login server URL
   - Example: `myregistry.azurecr.io`
   - Get it from: Azure Portal → Container Registry → Overview → Login server

2. **ACR_USERNAME**: Azure Container Registry username
   - Example: `myregistry`
   - Get it from: Azure Portal → Container Registry → Access keys → Username

3. **ACR_PASSWORD**: Azure Container Registry password
   - Get it from: Azure Portal → Container Registry → Access keys → Password

4. **ENV**: Contents of your environment configuration file
   - This should contain all the environment variables needed by the application
   - See `src/env_sample.txt` for the expected format
   - **Important**: Never commit your actual .env file to the repository

### How to Add Secrets

1. Go to your GitHub repository
2. Click on "Settings" → "Secrets and variables" → "Actions"
3. Click "New repository secret"
4. Add each of the secrets listed above

### Workflow Triggers

The workflow runs automatically when:
- Changes are pushed to the `main` branch AND
- Files in the `src/` directory or the workflow file itself are modified

You can also trigger the workflow manually from the Actions tab using "workflow_dispatch".

### How It Works

1. **Checkout**: Gets the latest code from the repository
2. **Login to ACR**: Authenticates with Azure Container Registry using the provided credentials
3. **Setup Docker Buildx**: Configures Docker BuildKit for advanced build features
4. **Build and Push**:
   - Builds the Docker image from `src/Dockerfile`
   - Uses BuildKit secrets to securely inject the ENV configuration
   - Tags the image with both the commit SHA and "latest"
   - Pushes the image to Azure Container Registry
   - Utilizes layer caching to speed up subsequent builds

### Security Notes

- The ENV secret is passed to Docker using BuildKit secrets, which prevents it from appearing in build logs or intermediate layers
- However, the .env file is created in the final image for runtime use by the application
- For enhanced security in production, consider using runtime secret injection via Azure Key Vault or container environment variables
- The workflow uses explicit minimal permissions (contents: read)
- Concurrency control ensures only one deployment runs at a time

### Troubleshooting

**Cache warnings on first run**: The first time the workflow runs, you may see warnings about the build cache not being found. This is expected and will be resolved after the first successful build.

**Authentication failures**: Verify that your ACR credentials are correct and that the "Admin user" is enabled in your Azure Container Registry settings.

**Missing ENV secret**: If the ENV secret is not configured, the Docker build will fail with an error message: "ERROR: env_content secret not provided"
