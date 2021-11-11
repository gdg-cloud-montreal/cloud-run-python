# Serverless Python Hello World with Cloud Run and Cloud Build

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

Pull the repo with DockerFile:

```
git pull 
```


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