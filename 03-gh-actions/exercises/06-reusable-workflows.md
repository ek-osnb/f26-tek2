# GitHub Action: Reusable Workflows

## Goal
Make the java test and docker CI workflows reusable so they can be called from other workflows. See the [GitHub documentation](https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows) for more details.

## Step 1: Create reusable workflow files
1. Inside the `.github/workflows` directory, create a new file named `java-test-reusable.yml`.
2. Open the `java-test-reusable.yml` file and add the following content:

```yaml
name: Java test reusable
on:
  workflow_call:

jobs:
  test:
    runs-on: ubuntu-latest
		permissions:
			contents: read
    steps:
      - name: Checkout code
        uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6.0.2
      - name: Set up JDK
        uses: actions/setup-java@be666c2fcd27ec809703dec50e508c2fdc7f6654 # v5.2.0
        with:
            java-version: '25'
            distribution: 'temurin'

      - name: Run tests
        run: mvn -B test
```
The `on: workflow_call` event allows this workflow to be called by other workflows.

For the `docker-ci` workflow, create a new file named `docker-ci-reusable.yml` and add the following content:

```yaml
name: Docker CI reusable
on:
  workflow_call:

jobs:
	build:
		runs-on: ubuntu-latest
		permissions:
			contents: read
		steps:
			- name: Checkout code
				uses: actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6.0.2
			- name: Test container
				run: |
					# Run the container in detached mode
					docker compose up -d
					
					# Wait for a few seconds to ensure the container is up and running
					sleep 40

					# Test the application endpoint
					curl -f http://localhost:8090/actuator/health || exit 1

					# Stop and remove the container
					docker compose down
```

## Step 2: Disable previous workflows
Move the existing `java-test.yml` and `docker-ci.yml` files to a new folder named `.github/disabled-workflows`. This ensures that they are not run, as only workflows in the `.github/workflows` folder are executed.


## Step 3: Update existing workflows to call reusable workflows
Now, create a new workflow file named `ci.yml` in the `.github/workflows` directory and add the following content:

```yaml
name: CI

on:
	pull_request:
		branches:
			- main
	workflow_dispatch:

jobs:
	java-test:
		uses: ./.github/workflows/java-test-reusable.yml

	docker-ci:
		needs: java-test
		uses: ./.github/workflows/docker-ci-reusable.yml
```

This `ci.yml` workflow calls the reusable `java-test-reusable.yml` and `docker-ci-reusable.yml` workflows. The `docker-ci` job depends on the successful completion of the `java-test` job.

## Step 4: Commit and Push
For the changes to take effect, you need to commit and push them to your repository. You can do this using the following commands:
```bash
git add .
git commit -m "Make Java test and Docker CI workflows reusable"
git push
```

## Step 5: Verify the Action
1. Go to your GitHub repository in your web browser.
2. Click on the "Actions" tab.
3. You should see a new workflow run for the "CI" action. Click on it to see the details.
4. Click on the jobs to see the logs. You should see the steps for checking out the code, setting up JDK, running tests, building the Docker image,running the container, and testing the application endpoint.
5. Ensure that both jobs complete successfully.
