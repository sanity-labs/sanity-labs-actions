# sanity-labs/sanity-labs-actions

Reusable GitHub Actions for the sanity-labs org. These composite actions handle building, pushing, and deploying containerized apps to the labs GKE cluster.

## Actions

### `setup-gcp`

Authenticates to GCP and logs in to Artifact Registry. Used internally by `build-and-push`, but can also be used standalone.

Requires the `GCR_CREDENTIALS` environment variable (org secret containing a GCP service account JSON key).

```yaml
- uses: sanity-labs/sanity-labs-actions/setup-gcp@main
  env:
    GCR_CREDENTIALS: ${{ secrets.GCR_CREDENTIALS }}
```

### `build-and-push`

Builds a Docker image and pushes it to `europe-docker.pkg.dev/sanity-artifacts/containers/`. Handles GCP auth automatically.

```yaml
- uses: sanity-labs/sanity-labs-actions/build-and-push@main
  env:
    GCR_CREDENTIALS: ${{ secrets.GCR_CREDENTIALS }}
```

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `image` | No | Repo name | Image name (without registry prefix) |
| `tag` | No | `github.sha` | Image tag |
| `dockerfile` | No | `Dockerfile` | Path to Dockerfile |
| `context` | No | `.` | Docker build context |
| `build-args` | No | | Newline-separated build args |

### `deploy`

Updates the `newTag` field in a kustomization.yaml overlay and pushes the change, triggering ArgoCD auto-sync.

```yaml
- uses: sanity-labs/sanity-labs-actions/deploy@main
  with:
    image-tag: ${{ github.sha }}
```

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `image-tag` | Yes | | The new image tag |
| `overlay-path` | No | `deploy/overlays/labs` | Path to kustomize overlay |

## Full Example

```yaml
name: Build and Deploy
on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4

      - uses: sanity-labs/sanity-labs-actions/build-and-push@main
        env:
          GCR_CREDENTIALS: ${{ secrets.GCR_CREDENTIALS }}

      - uses: sanity-labs/sanity-labs-actions/deploy@main
        with:
          image-tag: ${{ github.sha }}
```

## Setup

### GCP Auth

These actions authenticate using a GCP service account JSON key stored as `GCR_CREDENTIALS` at the org level.

To set up:

1. Create a service account in the `sanity-labs` GCP project
2. Grant `roles/artifactregistry.writer` on the `sanity-artifacts` project
3. Generate a JSON key
4. Add it as `GCR_CREDENTIALS` org secret in the sanity-labs GitHub org
