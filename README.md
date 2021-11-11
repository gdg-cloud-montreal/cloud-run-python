# Serverless Python Hello World with Cloud Run

This tutorial walks you through deploying Python `Hello World` webapp on [Cloud Run](https://cloud.google.com/run), Google Cloud's container based Serverless compute platform.

## What's under hood?

We will explore ways to build docker image with:
  * Dockerfile
  * Cloud Native [BuildPacks] without using Dockerfile

We will deploy application on Cloud Run:
  * Manually
  * Automted with Cloud Build Trigger from Github using cloudbuild.yaml and store Docker Images in GCR as well as Artifact Registry


## Tutorial

### Manually Deploying application on Cloud Run with Docker Image build via DockerFile

To streamline the rest of the tutorial define the key configuration settings and assign them to environment variables:

```
PROJECT_ID=$(gcloud config get-value project)
```
> `PROJECT_ID` holds the project id generated at the start of the tutorial.


Enable the necessary APIs for Cloud Run, Cloud Build APIs:

```
gcloud services enable --async \
  run.googleapis.com \
  cloudbuild.googleapis.com \
  containerregistry.googleapis.com \
  artifactregistry.googleapis.com
```

Pull the repo with python webapp:

```
git pull https://github.com/gdg-cloud-montreal/cloud-run-python
cd cloud-run-python
ls
```


Review the `app.py` python application:

```
cat app.py
```


Output: 

```
from flask import Flask, request

app = Flask(__name__)


@app.route("/", methods=["GET"])
def hello():
    """ Return a friendly HTTP greeting. """
    who = request.args.get("who", "World")
    return f"Hello {who}!\n"


if __name__ == "__main__":
    # Used when running locally only. When deploying to Cloud Run,
    # a webserver process such as Gunicorn will serve the app.
    app.run(host="localhost", port=8080, debug=True)
```


Review the `Dockerfile` for python application:

```
cat Dockerfile
```


```
# Use an official lightweight Python image.
# https://hub.docker.com/_/python
FROM python:3.9-slim

# Install production dependencies.
RUN pip install Flask gunicorn

# Copy local code to the container image.
WORKDIR /app
COPY . .

# Service must listen to $PORT environment variable.
# This default value facilitates local development.
ENV PORT 8080

# Run the web service on container startup. Here we use the gunicorn
# webserver, with one worker process and 8 threads.
# For environments with multiple CPU cores, increase the number of workers
# to be equal to the cores available.
CMD exec gunicorn --bind 0.0.0.0:$PORT --workers 1 --threads 8 --timeout 0 app:app
```


*Note:* You can define any container image conforming to the Cloud Run [Container Contract](https://cloud.google.com/run/docs/reference/container-contract)


Define the `DOCKER_IMG` environment variables:


```
DOCKER_IMG="gcr.io/$PROJECT_ID/helloworld-python"
echo $DOCKER_IMG
```

Now, build your container image using Cloud Build, by running the following command from the directory containing the Dockerfile:


```
gcloud builds submit --tag $DOCKER_IMG
```

!!! result
    Once pushed to the registry, you will see a `SUCCESS` message containing the image name. The image is stored in Container Registry and can be re-used if desired.


List all the container images associated with your current project using this command:

```
gcloud container images list
```


Before deploying, run and test the application locally from Cloud Shell, you can start it using these standard docker commands:


```
docker pull $DOCKER_IMG
docker run -p 8080:8080 $DOCKER_IMG
```

Deploy your containerized application to Cloud Run with the following command:

```
gcloud run deploy helloworld-python \
  --image $DOCKER_IMG \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated
```


### Manually Deploying application on Cloud Run with Docker Image build via BuildPacks

[To DO]

### Automatically Deploying application on Cloud Run using Cloud Build Trigger

#### Enable Required IAM permissions to use Cloud Run with Cloud Build:


```
Step 1 Open the Cloud Build settings page in the Cloud Console

Step 2 In the Service account permissions panel, set the status of the Cloud Run Admin role to ENABLED

Step 3 In the Additional steps may be required pop-up, click GRANT ACCESS TO ALL SERVICE ACCOUNTS
```

#### Review cloudbuild.yaml in the github repo

```
cd cloud-run-python
cat cloudbuild.yaml
```

```
steps:
# Build the container image
- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', 'gcr.io/$PROJECT_ID/python-hello', '.']
# Push the container image to Container Registry
- name: 'gcr.io/cloud-builders/docker'
  args: ['push', 'gcr.io/$PROJECT_ID/python-hello']
# Deploy container image to Cloud Run
- name: 'gcr.io/cloud-builders/gcloud'
  args: ['run', 'deploy', '${_APP_NAME}', '--image', 'gcr.io/$PROJECT_ID/python-hello', '--region', '${_REGION}', '--platform', 'managed', '--allow-unauthenticated']
images:
- gcr.io/$PROJECT_ID/python-hello
substitutions:
    _APP_NAME: hello-world
    _REGION: us-central1
```



#### Enable CI/CD by configuring a Cloud Build trigger

Define the `PROJECT_ID` variable to hold the current project ID

```
PROJECT_ID=$(gcloud config get-value project)
```

Connect Github Repository to Cloud Build:

```
Follow steps:  # https://cloud.google.com/cloud-build/docs/automating-builds/create-github-app-triggers#installing_the_cloud_build_app
```


Create a trigger to activate Cloud Build when a new commit to the source repo occurs:


```
export REPO_NAME=cloud-run-python
export REPO_OWNER=gdg-cloud-montreal
```

```
gcloud beta builds triggers create github \
    --repo-name=$REPO_NAME \
    --repo-owner=$REPO_OWNER \
    --branch-pattern="main" \
    --build-config=cloudbuild.yaml
```

*Note:* Make sure to use the proper path in the cloudbuild.yaml file.