# GitHub actions: Push docker image to GitHub Container Registry (GHCR)

## Goal
The goal of this exercise is to create a GitHub Action that builds a Docker image from a Dockerfile and pushes it to the GitHub Container Registry (GHCR).

## Step 1: Create a new workflow file
Inside the `.github/workflows` directory, create a new file named `push-ghcr.yml`.

Open the `push-ghcr.yml` file and add the following content:

```yaml
name: Push to GHCR

on:
  push:
    branches: [ main ]
  workflow_dispatch:

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # Using the reusable workflow for Java tests
  java-test:
    uses: ./.github/workflows/java-test-reusable.yml

	docker-ci:
		needs: java-test
		uses: ./.github/workflows/docker-ci-reusable.yml

  # Build & push Docker image (after tests pass)
  build-and-push:
    needs: docker-ci
    runs-on: ubuntu-latest
    permissions:
      contents: read          # needed for checkout
      packages: write         # needed to push to GHCR
    steps:
      - name: Checkout code
        uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6.0.2

      - name: Log in to GitHub Container Registry
        uses: docker/login-action@c94ce9fb468520275223c153574b00df6fe4bcc9 # v3.7.0
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata (tags, labels)
        id: meta
        uses: docker/metadata-action@c299e40c65443455700f0fdfc63efafe5b349051 # v5.10.0
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=sha,prefix={{branch}}-
            type=raw,value=latest,enable={{is_default_branch}}

      - name: Build and push
        uses: docker/build-push-action@601a80b39c9405e50806ae38af30926f9d957c47 # v6.19.1
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
```

Note that we are using the reusable workflow `java-test-reusable.yml` to run the Java tests before building and pushing the Docker image. This ensures that the image is only built and pushed if the tests pass.

In the `Log in to GitHub Container Registry` step, we use the `docker/login-action` to authenticate with GHCR using the `GITHUB_TOKEN` secret, which is automatically provided by GitHub Actions (we do not need to create it manually).

The `Extract metadata` step uses the `docker/metadata-action` to generate tags and labels for the Docker image based on the Git reference (branch or tag) that triggered the workflow. An example of tags that will be created are:
- `latest` (if the push is to the default branch, usually `main`)
- `main-<short-sha>` (for pushes to the `main` branch)
- `<tag-name>` (for pushes to tags)

In our example repository `your-username/your-repo`, the image will be pushed to `ghcr.io/your-username/your-repo`.

The `Build and push` step uses the `docker/build-push-action` to build the Docker image from the Dockerfile in the repository and push it to GHCR with the generated tags and labels.

## Step 2: Commit and Push
For the action to take effect, you need to commit and push the changes to your repository.
You can do this using the following commands:
```bash
git add .
git commit -m "Add GitHub Action to push Docker image to GHCR"
git push
```

## Step 3: Verify the Action
1. Go to your GitHub repository in your web browser.
2. Click on the "Actions" tab.
3. You should see a new workflow run for the "Push to GHCR" action.
4. Click on it to see the details.
5. Click on the jobs to see the logs. You should see the steps for checking out the code, running tests, logging in to GHCR, extracting metadata, and building and pushing the Docker image.
6. If everything is set up correctly, the Docker image should be pushed to the GitHub Container Registry. You can verify this by going to the "Packages" section of your repository or your GitHub profile.
