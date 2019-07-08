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

## Create a Service Account
Create service account. Following [this](https://cloud.google.com/iam/docs/creating-managing-service-accounts#iam-service-accounts-create-gcloud) step. The service account is used to authenticate an application to use GCP APIs.

```bash
gcloud beta iam service-accounts create pubsub-sa-1 --description "learning pubsub" --display-name "pubsub-sa-1"
```

## Create a Service Account Key
Create service account key. Following [this](https://cloud.google.com/iam/docs/creating-managing-service-account-keys) step.

```bash
gcloud iam service-accounts keys create ~/key.json --iam-account [SA-NAME]@[PROJECT-ID].iam.gserviceaccount.com
```

The `iam-account` value is the `EMAIL` from command: `gcloud iam service-accounts list`.

Download the key.json file to your local computer. Then create environment variable, `GOOGLE_APPLICATION_CREDENTIALS` , with value as the full path to the key.json. If the environment variable is not set, GCP client libraries employ a strategy for finding the credentials. Click [here](https://cloud.google.com/docs/authentication/production#providing_credentials_to_your_application) to learn more. 

## Add Role to Service Account
Add role to service account. Following steps: [1](https://cloud.google.com/dlp/docs/auth), [2](https://cloud.google.com/iam/docs/granting-roles-to-service-accounts).

```bash
gcloud projects add-iam-policy-binding my-project-123 \
  --member serviceAccount:my-sa-123@my-project-123.iam.gserviceaccount.com \
  --role roles/editor
```

## Quickstart using gcloud CLI
Use the CLI. Following [this](https://cloud.google.com/pubsub/docs/quickstart-cli) quickstart tutorial. This is a simple demonstration of creating a topic, subscription, followed by message publish and receive.

```bash
# create topic
gcloud pubsub topics create my-topic

# confirm
gcloud pubsub topics list

# create subscription
gcloud pubsub subscriptions create --topic my-topic my-subscription
# From man page: "Creates one or more Cloud Pub/Sub subscriptions for a given topic. The new subscription defaults to a PULL subscription unless a PUSH endpoint is specified."

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

## Sample Program

### Node.js

Following the docs for [Node.js Pub/Sub client library](https://github.com/googleapis/nodejs-pubsub).

#### Create a new project
```bash
# start new Node.js project (follow prompts)
npm init

# add client library
npm install @google-cloud/pubsub

# create a Javascript file
touch HelloPubSub.js
```

#### Publish to Topic
```javascript
async function demoPublish() {
    projectId = 'my-project';
    topicName = 'my-topic';

    // import the pubsub client library
    const { PubSub } = require('@google-cloud/pubsub');

    // create client
    const pubsub = new PubSub({projectId});

    // create data
    helloWorld = {
            "msg": "hello Node.js pub/sub"
        };

    // publish data
    const messageId = await pubsub.topic(topicName).publishJSON(helloWorld);
    console.log(`Message ${messageId} published.`)
}
```

#### Receive Message From Topic
Note that the messages are received via a topic subscription which is assumed to exist by name `my-subscription`. See the section above named "Quickstart using gcloud CLI".
```javascript
async function demoReceive() {
    const timeout = 10;
    projectId = 'my-project';
    topicName = 'my-topic';

    // Receive the message via the subscription
    //   https://github.com/googleapis/nodejs-pubsub/blob/master/samples/subscriptions.js

    // authentication is required and can be supplied using the environment variable: GOOGLE_APPLICATION_CREDENTIALS
    const pubsub = new PubSub({projectId});

    // obtain handle to the subscription
    const subscription = pubsub.subscription("my-subscription");

    // create message handler (and use it later)
    let messageCount = 0;
    const messageHandler = (message) => {
        console.log(`Received message ${message.id}:`);
        console.log(`\tData: ${message.data}`);
        console.log(`\tAttributes: ${message.attributes}`);
        messageCount += 1;

        // Acknowledge the message
        message.ack();
    }

    // listen for messages via the subscription
    subscription.on(`message`, messageHandler);

    setTimeout(() => {
        subscription.removeListener(`message`, messageHandler);
        console.log(`${messageCount} messages(s) received.`);
    }, timeout * 1000)
}
```
