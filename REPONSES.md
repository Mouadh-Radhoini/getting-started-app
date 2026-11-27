# Exam TP 10 — Answers

## Decision 1 — Why use two Docker Compose directories?

We use two directories, `recette` and `prod`, to separate the testing and production Docker Compose projects.

By default, Docker Compose uses the project directory name to name containers, networks, and volumes.

Therefore, the `recette_donnees` and `prod_donnees` volumes are independent.

If both Compose files were located in the same directory and executed using the `-f` option without specifying different project names, Docker Compose could use the same project name, causing conflicts or replacing existing containers.

The `-p` option also allows us to explicitly define different project names.

**Verification commands:**

- `docker compose ls`
- `docker ps`
- `docker volume ls`

## Decision 2 — Why use a second manual Jenkins job?

The `cd-taches` job automatically builds the Docker image, deploys it to the testing environment, and executes the tests.

The `valide` tag is assigned and published only if all tests succeed.

The `prod-taches` job is manually triggered by the responsible person after verifying the test results. It downloads the validated Docker image without rebuilding it.

This separates automatic testing from the decision to deploy the application to production.

It also ensures that the image deployed in production is the same image that was successfully tested.

**Verification commands:**

- `docker image ls mouadhradhouani/taches`
- `docker image inspect mouadhradhouani/taches:valide --format '{{.Id}}'`
- `curl http://localhost:3000/items`

## Decision 3 — Why use down -v only in the testing environment?

The data in the testing environment is temporary. The `docker compose down -v` command removes the containers, networks, and associated non-external volumes.

In production, this command must not be used because it would delete the volume containing the MySQL database data.

Instead, we use `docker compose up -d` to update the production application while preserving the existing database data.

If the tests fail, the Validation and Cleanup stages are not executed. The testing containers and volumes remain available for troubleshooting and diagnosis.

This approach protects production data while allowing the testing environment to be cleaned after successful validation.

**Verification commands:**

- `docker volume ls`
- `docker ps`
- `curl http://localhost:3000/items`
