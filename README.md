## GitHub Workflows

### [Docker Build and Publish workflow](.github/workflows/docker-publish.yml)

This workflow builds docker iamge and push to GitHub Container Registry using GitHub Actions.

- Make sure you have added `GHCR_TOKEN` as a repository secret in Repo Settings. [See Doc for how to add Repository Secrets](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-a-repository)
- This workflow assumes that you have added a `Dockerfile` in the root directory.
- You will be required to manually trigger the workflow from the **Actions** tab.
- To build the Docker image automatically on push to main, update [docker-publish.yml](.github/workflows/docker-publish.yml) with following:
```aiignore
on:
  push:
    branches: [ main ]
```

### [Helm Deploy workflow](.github/workflows/helm-deploy.yml)

This workflow deploys your Helm chart to an EKS cluster using GitHub Actions.

- Following Secrets must be set in Settings → Secrets and Variables → Actions → Repository secrets.

  | Secret Name                    | Description                                                       |
  |-------------------------------|-------------------------------------------------------------------|
  | `IAM_ROLE_ARN`                | IAM Role ARN for GitHub OIDC federation                          |
  | `AWS_REGION`                  | AWS region of your EKS cluster                                   |
  | `CLUSTER_NAME`                | Name of your EKS cluster                                         |
  | `TWINGATE_SERVICE_KEY_SECRET_NAME` | Twingate connector service key                             |
  | `ENV_SECRET_JSON`             | JSON of environment variables (e.g., `{"KEY":"val"}`)             |
  | `GHCR_TOKEN`                  | GitHub token or Personal Access Token (PAT) for accessing GHCR   |

#### Example `ENV_SECRET_JSON`

Store the following as a **single-line JSON string** in your `ENV_SECRET_JSON` GitHub Secret:

```json
{
  "NODE_ENV": "production",
  "API_KEY": "your-api-key-here",
  "SENTRY_DSN": "https://example@sentry.io/12345"
}
```

> 🛠️ **Note:** The following secrets will be provided by the **DevOps team**:
> - `IAM_ROLE_ARN`
> - `AWS_REGION`
> - `CLUSTER_NAME`
> - `TWINGATE_SERVICE_KEY_SECRET_NAME`

### How to Run the Helm Deploy Workflow

You can manually trigger the workflow from the **Actions** tab in GitHub by selecting the **Helm Deploy to EKS** workflow and clicking **Run workflow**. When prompted, provide the following inputs:

| Input           | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| `namespace`     | Kubernetes namespace to deploy to                                           |
| `application`   | Application name (used for Helm release and Kubernetes secret names)       |
| `repository`    | Full Docker image repository URL (e.g., `ghcr.io/your-name/app`)           |
| `tag`           | Docker image tag to deploy                                                 |
| `containerPort` | Port your container listens on (default: `80`)                             |
| `domain`        | Domain name used for ingress (e.g., `app.example.com`)                     |
| `ingressClass`  | Ingress class name (`internal-nginx`, `external-nginx`, etc.)              |
| `healthPath`    | Path for readiness and liveness probes (default: `/healthz`)               |

> 📝 Example: To deploy an app to the `dev` namespace with image `ghcr.io/my-org/my-app:1.0.3`, listening on port `8080`, with ingress configured for `app.dev.example.com`, you'd provide the inputs accordingly when running the workflow manually.

> 📌 **Note:**
> - For applications deployed using **`internal-nginx`** ingress, you must **add the application domain as a resource in the Twingate Admin Console** in order to access it over VPN.
> - Applications deployed with **`external-nginx`** ingress will be accessible publicly by default (no Twingate configuration needed).