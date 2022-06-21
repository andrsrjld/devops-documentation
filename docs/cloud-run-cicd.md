## DEPLOY CLOUD RUN USING GITLAB CI

In this article, I would guide through deploying serverless containerized applications to [Cloud Run](https://cloud.google.com/run), using [GitLab CI](https://docs.gitlab.com/ee/ci/) and [Cloud Build](https://cloud.google.com/cloud-build).

> Cloud Run is a managed compute platform that enables you to run stateless serverless containers that automatically scales.
> 
> Cloud Build is a service that executes your builds on Google Cloud Platform infrastructure.
> 
> GitLab CI service is a part of GitLab that build and test the software whenever developer pushes code to application repo.

## Prerequisites
* Create a Google Cloud Platform (GCP) project, or use an existing one.
* Create a GitLab Repo.
* Enable the Cloud Run API.
* Enable the Cloud Build API.
* Clone the sample codes or setup your own codes with a Dockerfile.

## Creating a Service Account for Google Cloud Build
* On Google Cloud, navigate through Cloud Build > Settings.
* Under Service account permissions, ensure that Cloud Run & Service Accounts are ENABLED , this allows you deploy to Cloud Run.

![](https://miro.medium.com/max/862/1*lq253WALRoudTmG-80DdoA.png)

* Since I have given Cloud Build sufficient permissions, I can create a Cloud Build service account on IAM & Admin > Service Accounts. I’ll create a service account (NAME@PROJECT.iam.gserviceaccount.com) and give it the Cloud Build Service Agent. On the created service account page, click on Add Key > JSON.

![](https://miro.medium.com/max/1400/1*qP2UdBWXH7cFKe910H6K8A.png)

## Configure GitLab CI to use Service Accounts
On the GitLab repo, navigate through Setting > CI/CD > Variables.

![](https://miro.medium.com/max/1400/1*Hxu_w-yRfu8sht8QUGgwzA.png)

As seen above, I created a variable for GCP_PROJECT_ID whose value is the Google Cloud Project ID and GCP_SERVICE_KEY whose value is the contents of the JSON service account earlier created.

## Continuous Deployment to Cloud Run

With just some few steps left, my application would be continuously deployed to Cloud Run directly from our GitLab repo.

My application also has a Dockerfile which is configured to run on port 8080 (the default port for Cloud Run).

Finally, I created a cloudbuild.yaml file which contains the commands to build & deploy by Cloud Build and .gitlab-ci.yml file which triggers the deployment processes when code is pushed.

Here’s a preview of my Cloud Build CI file:

    steps:
        # build the container image
      - name: 'gcr.io/cloud-builders/docker'
        args: [ 'build', '-t', 'gcr.io/$PROJECT_ID/demo-app', '.' ]
        # push the container image
      - name: 'gcr.io/cloud-builders/docker'
        args: [ 'push', 'gcr.io/$PROJECT_ID/demo-app']
        # deploy to Cloud Run
      - name: "gcr.io/cloud-builders/gcloud"
        args: ['run', 'deploy', 'erp-ui', '--image', 'gcr.io/$PROJECT_ID/demo-app', '--region', 'europe-west4', '--platform', 'managed', '--allow-unauthenticated']

Here’s a preview of my GitLab CI file:

    image: docker:latest

    stages:
      - deploy

    deploy:
      stage: deploy
      image: google/cloud-sdk
      services:
        - docker:dind
      script:
        - echo $GCP_SERVICE_KEY > gcloud-service-key.json # Google Cloud service accounts
        - gcloud auth activate-service-account --key-file gcloud-service-key.json
        - gcloud config set project $GCP_PROJECT_ID
        - gcloud builds submit . --config=cloudbuild.yaml


