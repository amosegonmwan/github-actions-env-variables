# Deployment Workflow for React NodeJS Project

This GitHub Actions workflow is designed to automate the testing and deployment process for a React NodeJS project. The workflow includes a `test` job and a `deploy` job that run on every push to the `main` and `dev` branches. The workflow uses environment variables for configuration, and secrets are utilized for sensitive information.

## Workflow Details:

```yaml
name: Deployment
on:
  push:
    branches:
      - main
      - dev
env:
  MONGODB_DB_NAME: gha-demo # Workflow-level environment variable

jobs:

  # Test job runs tests on the project
  test:
    env:
      MONGODB_CLUSTER_ADDRESS: cluster0.nnreln6.mongodb.net
      MONGODB_USERNAME: ${{ secrets.MONGODB_USERNAME }} 
      MONGODB_PASSWORD: ${{ secrets.MONGODB_PASSWORD }}
      PORT: 8080
    runs-on: ubuntu-latest
    steps:
      - name: Get Code
        uses: actions/checkout@v3
      - name: Cache dependencies
        uses: actions/cache@v3
        with:
          path: ~/.npm
          key: npm-deps-${{ hashFiles('**/package-lock.json') }}
      - name: Install dependencies
        run: npm ci
      - name: Run server
        run: npm start & npx wait-on http://127.0.0.1:$env:PORT
      - name: Run tests
        run: npm test
      - name: Output information
        run: |
          echo "MONGODB_USERNAME: ${{ env.MONGODB_USERNAME }}"

  # Deploy job depends on the success of the test job
  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Output information
        run: |
          echo "MONGODB_DB_NAME: ${{ env.MONGODB_DB_NAME }}"
```

## Workflow Explanation:
### Environment Variables:
MONGODB_DB_NAME: Workflow-level environment variable set to "gha-demo."
MONGODB_CLUSTER_ADDRESS, MONGODB_USERNAME, MONGODB_PASSWORD, PORT: Environment variables for the test job, allowing dynamic configuration. Secrets are used for sensitive information.

### Test Job:
* Utilizes the testing environment (commented out).
* Runs on an Ubuntu environment.
* Checks out the code, caches npm dependencies, and installs them.
* Runs the server, waits for it to be ready, and then executes tests.
* Outputs the value of the MONGODB_USERNAME environment variable.

### Deploy Job:
* Depends on the success of the test job.
* Runs on an Ubuntu environment.
* Outputs the value of the MONGODB_DB_NAME environment variable.
