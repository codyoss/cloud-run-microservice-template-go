# Cloud Run Template Microservice

A template repository for a Cloud Run microservice, written in Go.

[![Run on Google Cloud](https://deploy.cloud.run/button.svg)](https://deploy.cloud.run)

## Prerequisite

* Enable the Cloud Run API via the [console](https://console.cloud.google.com/apis/library/run.googleapis.com?_ga=2.124941642.1555267850.1615248624-203055525.1615245957) or CLI:

```bash
gcloud services enable run.googleapis.com
```

## Features

* **gorilla/mux**: A request router and dispatcher
* **Buildpack support** Tooling to build production-ready container images from source code and without a Dockerfile
* **Dockerfile**: Container build instructions
* **SIGTERM handler**: Catch termination signal for cleanup before Cloud Run stops the container
* **Service metadata**: Access service metadata, project Id and region, at runtime
* **Structured logging w/ Log Correlation** JSON formatted logger, parsable by Cloud Logging, with [automatic correlation of container logs to a request log](https://cloud.google.com/run/docs/logging#correlate-logs).
* **Unit and System tests** Basic unit and system tests setup for the microservice

## Local Development

### Cloud Code

This template works with [Cloud Code](https://cloud.google.com/code), an IDE extension
to let you rapidly iterate, debug, and run code on Kubernetes and Cloud Run.

Learn how to use Cloud Code for:

* Local development - [VSCode](https://cloud.google.com/code/docs/vscode/developing-a-cloud-run-service), [IntelliJ](https://cloud.google.com/code/docs/intellij/developing-a-cloud-run-service)

* Local debugging - [VSCode](https://cloud.google.com/code/docs/vscode/debugging-a-cloud-run-service), [IntelliJ](https://cloud.google.com/code/docs/intellij/debugging-a-cloud-run-service)

* Deploying a Cloud Run service - [VSCode](https://cloud.google.com/code/docs/vscode/deploying-a-cloud-run-service), [IntelliJ](https://cloud.google.com/code/docs/intellij/deploying-a-cloud-run-service)
* Creating a new application from a custom template (`.template/templates.json` allows for use as an app template) - [VSCode](https://cloud.google.com/code/docs/vscode/create-app-from-custom-template), [IntelliJ](https://cloud.google.com/code/docs/intellij/create-app-from-custom-template)

### CLI tooling

#### Local development

1. Set Project Id:

    ```bash
    export GOOGLE_CLOUD_PROJECT=<GCP_PROJECT_ID>
    ```

2. Build and Start the server:

    ```bash
    go build -o server && ./server
    ```

#### Deploying a Cloud Run service

1. Set Project Id:

    ```bash
    export GOOGLE_CLOUD_PROJECT=<GCP_PROJECT_ID>
    ```

2. Use the gcloud credential helper to authorize Docker to push to your
   Container Registry:

   ```bash
    gcloud auth configure-docker
    ```

3. Build and push the container using docker:

    ```bash
    docker build . -t gcr.io/$GOOGLE_CLOUD_PROJECT/microservice-template
    docker push gcr.io/$GOOGLE_CLOUD_PROJECT/microservice-template
    ```

4. Deploy to Cloud Run:

    ```bash
    gcloud run deploy microservice-template \
      --image gcr.io/$GOOGLE_CLOUD_PROJECT/microservice-template \
    ```

### Run sample tests

1. [Pass credentials via `GOOGLE_APPLICATION_CREDENTIALS` env var](https://cloud.google.com/docs/authentication/production#passing_variable):

    ```bash
    export GOOGLE_APPLICATION_CREDENTIALS="[PATH]"
    ```

2. Set Project Id:

    ```bash
    export GOOGLE_CLOUD_PROJECT=<GCP_PROJECT_ID>
    ```

3. Run unit tests

    ```bash
    go test ./...
    ```

4. Run system tests

    ```bash
    gcloud builds submit \
        --config advance.cloudbuild.yaml \
        --substitutions=COMMIT_SHA=manual,REPO_NAME=manual
    ```

    The Cloud Build configuration file will build and deploy the containerized
    service to Cloud Run, run tests, then clean up testing resources. This
    configuration restricts public access to the test service. Therefore,
    service accounts need to have the permission to issue ID tokens for request
    authorization:
    * Enable Cloud Build and IAM APIs:

        ```bash
        gcloud services enable cloudbuild.googleapis.com iamcredentials.googleapis.com
        ```

    1. Set environment variables.

        ```bash
        export PROJECT_ID="$(gcloud config get-value project)"
        export PROJECT_NUMBER="$(gcloud projects describe $(gcloud config get-value project) --format='value(projectNumber)')"
        ```

    2. Create service account `token-creator` with `Service Account Token Creator` and `Cloud Run Invoker` roles.

        ```bash
        gcloud iam service-accounts create token-creator

        gcloud projects add-iam-policy-binding $PROJECT_ID \
            --member="serviceAccount:token-creator@$PROJECT_ID.iam.gserviceaccount.com" \
            --role="roles/iam.serviceAccountTokenCreator"
        gcloud projects add-iam-policy-binding $PROJECT_ID \
            --member="serviceAccount:token-creator@$PROJECT_ID.iam.gserviceaccount.com" \
            --role="roles/run.invoker"
        ```

    3. Add `Service Account Token Creator` role to the Cloud Build service account.

        ```bash
        gcloud projects add-iam-policy-binding $PROJECT_ID \
            --member="serviceAccount:$PROJECT_NUMBER@cloudbuild.gserviceaccount.com" \
            --role="roles/iam.serviceAccountTokenCreator"
        ```

## Maintenance & Support

This repo performs basic periodic testing for maintenance. Please use the issue tracker for bug reports, features requests and submitting pull requests.

## Contributions

Please see the [contributing guidelines](CONTRIBUTING.md)

## License

This library is licensed under Apache 2.0. Full license text is available in [LICENSE](LICENSE).
