# About
The Google Cloud Platform has a service called [Cloud Pub/Sub](https://cloud.google.com/pubsub/), which "provides reliable, many-to-many, asynchronous messaging between applications". This readme documents key commmands to run via the Cloud Shell (browser-based shell available through GCP Console), then using `gloud` CLI.

The main idea:
* Application X subscribes to a topic
* Application Y publishes messages to a topic
* Applicaiton Xd receives messages via its topic subscription


# Setup

## Create a Project
Create a project. Following [this](https://cloud.google.com/resource-manager/docs/creating-managing-projects) step.

```bash
gcloud projects create PROJECT_ID
gcloud projects create PROJECT_ID --name=NAME

# arguments:
  # PROJECT_ID is the ID for the project you want to create. A project ID must start with a lowercase letter, and can contain only ASCII letters, digits, and hyphens, and must be between 6 and 30 characters.
  # NAME for the project you want to create. If not specified, will use project id as name.

# change the name
gcloud projects update PROJECT_ID --name=BetterProjectName

# set the project
gcloud config set project PROJECT_ID
```

## Quickstart using gcloud CLI
Use the CLI. Following [this](https://cloud.google.com/pubsub/docs/quickstart-cli) quickstart tutorial.

```bash
# create topic
gcloud pubsub topics create my-topic

# confirm
gcloud pubsub topics list

# create subscription
gcloud pubsub subscriptions create --topic my-topic my-subscription
# From man page: "Creates one or more Cloud Pub/Sub subscriptions for a given topic. The new subscription defaults to a PULL subscription unless a push endpoint is specified."

# confirm
gcloud pubsub subscriptions list

# publish a message
gcloud pubsub topics publish my-topic --message "hello pub/sub!"

# receive the message
gcloud pubsub subscriptions pull --auto-ack my-subscription

# output
┌────────────────┬─────────────────┬────────────┐
│      DATA      │    MESSAGE_ID   │ ATTRIBUTES │
├────────────────┼─────────────────┼────────────┤
│ hello pub/sub! │ 123456712345671 │            │
└────────────────┴─────────────────┴────────────┘
```

