# About
The Google Cloud Platform has a service called [App Engine](https://cloud.google.com/appengine/), which offers "zero server management or configuration deployments [so that] developers can focus on building highly scalable applications without the management overhead." AppEngine is a compute service offering from GCP. The comparable service in Amazon Web Services is Elastic Beanstalk. This readme documents introductory concepts, and the enclosed artifacts are used to demonstrate those concepts.

# Setup

## Create a Project
Create a project. Following [this](https://cloud.google.com/resource-manager/docs/creating-managing-projects) step.

```
gcloud projects create PROJECT_ID
gcloud projects create PROJECT_ID --name=NAME

# arguments:
  # PROJECT_ID is the ID for the project you want to create. A project ID must start with a lowercase letter, and can contain only ASCII letters, digits, and hyphens, and must be between 6 and 30 characters.
  # NAME for the project you want to create. If not specified, will use project id as name.

# change the name
gcloud projects update PROJECT_ID --name=BetterProjectName

# set the project

```

## Clone Sample Code
1. Clone this repo: `git clone https://github.com/GoogleCloudPlatform/golang-samples.git`

## Run Sample Code
1. Change directory: `cd golang-samples/appengine/go11x/helloworld`
1. Create an app: `gcloud app create` and follow menu prompts
1. Deploy the app: `gcloud app deploy`
1. Goto URL: https://[your_project_id].appspot.com, and see message: "Hello, World!"


## Disable and Delete Sample Code (Avoid Charges!)
1. In GCP Console (web UI), type "AppEngine" in search box to open App Engine's [dashboard](https://console.cloud.google.com/appengine).
1. Using left-hand menu, goto Settings page
1. Click button "Disable application"
1. In GCP Console, type "IAM", and goto "IAM & admin" dashboard.
1. Using left-hand menu, goto Manage Resources page
1. Select the checkbox next to the project
1. Click Delete





