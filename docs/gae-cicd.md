## Automatically deploy to Google App Engine with Gitlab CI
Automate the boring task 😁 of deployment

![](https://miro.medium.com/max/1400/1*Y9Pav9mnsRsDV6COhtsw6Q.png)

All we have to do is:

1. Create a service account (I assume you know about it)
2. Setup Gitlab CI
3. Magic happens!

## Creating a service account
Visit [iam-admin](https://console.cloud.google.com/iam-admin/serviceaccounts/create) page and create the service account with required permissions and create the key.

![](https://miro.medium.com/max/1086/1*l-uPDkJ1CDb7-G0B5kkV3Q.png)

1. App Engine Admin (This is required only if your using dispatch.yaml)
2. App Engine Deployer and App Engine Service Account (Ignore this both,if you have given 1st permission)
3. Grant `Service Account User` permission to CI/CD service account
4. Create the JSON key (You will need this in Gitlab configuration)

![](https://miro.medium.com/max/1150/1*QhlN9WfRMIAF_h0kGkSNCA.png)

Now visit [storage](https://console.cloud.google.com/storage/browser) and go to buckets:

1. staging.PROJECT-ID.appspot.com
2. us.artifacts.PROJECT-ID.appspot.com (Create bucket with this name, if you don’t have one)

Now add your service account as a member of this buckets and give permissions Storage Object Creator and Storage Object Viewer.

![](https://miro.medium.com/max/1400/1*E02Xr_7tnbt1Jxdro9Mo-Q.png)

One final step in Google Cloud Developer console.

Enable [cloud-build](https://console.cloud.google.com/cloud-build/builds) api and you will need to have billing account linked to enable this api.

## Setup Gitlab CI

We will deploy master branch to production and staging branch to staging.

Visit CI/CD settings of your gitlab project.

![](https://miro.medium.com/max/1400/1*4c_fcTpfarDB1On_5lAFIQ.png)

Let’s add two variable **PROJECT_ID** and **SERVICE_ACCOUNT**

**SERVICE_ACCOUNT**: Put the data of JSON key which we have downloaded while creating service account.

Now create the .gitlab-ci.yml file in your root folder.

    stages:
       - build
       - deploy

    build_project:
       stage: build
       image: node:16
       script:
          - echo "Start building App"
          - yarn install
          - yarn build
          - echo "Build successfully!"
       artifacts:
          expire_in: 1 hour
          paths:
           - build
           - node_modules

    deploy_production:
       stage: deploy
       image: google/cloud-sdk:alpine
       environment: Production
       only:
          - master
       script:
          - echo $SERVICE_ACCOUNT > /tmp/$CI_PIPELINE_ID.json
          - gcloud auth activate-service-account --key-file /tmp/$CI_PIPELINE_ID.json
          - gcloud --quiet --project $PROJECT_ID app deploy app.yaml
       after_script:
          - rm /tmp/$CI_PIPELINE_ID.json

    deploy_staging:
       stage: deploy
       image: google/cloud-sdk:alpine
       environment: Staging
       only:
          - develop
       script:
          - echo $SERVICE_ACCOUNT > /tmp/$CI_PIPELINE_ID.json
          - gcloud auth activate-service-account --key-file /tmp/$CI_PIPELINE_ID.json
          - gcloud --quiet --project $PROJECT_ID app deploy staging.yaml --verbosity=info
       after_script:
          - rm /tmp/$CI_PIPELINE_ID.json

this is an exaple of app.yaml

    runtime: nodejs16
    env: standard
    service: dashboard-production
    instance_class: F1
    handlers:
      - url: /(.*\..+)$
        static_files: build/\1
        require_matching_file: false
        upload: build/(.*\..+)$
      - url: /.*
        static_files: build/index.html
        require_matching_file: false
        upload: build/index.html
      - url: .*
        script: auto
    automatic_scaling:
      min_idle_instances: automatic
      max_idle_instances: automatic
      min_pending_latency: automatic
      max_pending_latency: automatic
    network: {}

and this is an example of staging.yaml

    runtime: nodejs16
    env: standard
    service: default
    instance_class: F1
    handlers:
      - url: /(.*\..+)$
        static_files: build/\1
        require_matching_file: false
        upload: build/(.*\..+)$
      - url: /.*
        static_files: build/index.html
        require_matching_file: false
        upload: build/index.html
      - url: .*
        script: auto
    automatic_scaling:
      min_idle_instances: automatic
      max_idle_instances: automatic
      min_pending_latency: automatic
      max_pending_latency: automatic
    network: {}

Changes of master and staging branches will deploy to production and staging respectively.
I have used dispatch.yaml file, if you don’t have one just remove it from the script as well.

Push the .gitlab-ci.yml in your repo and your are done.

Source code: https://github.com/mryogesh/yogesh
In case if you want to see staging-app.yaml and dispatch.yaml file.

Magic 🎩
Now, git push automatically deploys your code to staging and production.

![](https://miro.medium.com/max/1000/1*JnXiTdCOAdsSO7V84rH2Zg.gif)