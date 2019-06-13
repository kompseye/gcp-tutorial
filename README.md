# About
The Google Cloud Platform has a service called Data Loss Prevention (DLP). This readme documents key commands to run via the Cloud Shell (browser-based shell available through GCP Console UI), then using the `gcloud` CLI.

# Setup

Create service account. Following [this](https://cloud.google.com/iam/docs/creating-managing-service-accounts#iam-service-accounts-create-gcloud) step.

```bash
gcloud beta iam service-accounts create dlp-lesson-1 --description "lesson 1" --display-name "dlp-lesson-1"
```

Create service account key. Following [this](https://cloud.google.com/iam/docs/creating-managing-service-account-keys) step.

```bash
gcloud iam service-accounts keys create ~/key.json --iam-account [SA-NAME]@[PROJECT-ID].iam.gserviceaccount.com
```

The `iam-account` value is the `EMAIL` from command: `gcloud iam service-accounts list`

Add role to service account. Following steps: [1](https://cloud.google.com/dlp/docs/auth), [2](https://cloud.google.com/iam/docs/granting-roles-to-service-accounts).

```bash
gcloud projects add-iam-policy-binding my-project-123 \
  --member serviceAccount:my-sa-123@my-project-123.iam.gserviceaccount.com \
  --role roles/editor
```

Activate the service account

```bash
gcloud auth activate-service-account --key-file <path to key>
```

Obtain authorization token

```bash
gcloud auth print-access-token
```

Use the token with DLP API call

```bash
curl -s -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer [ACCESS_TOKEN]' \
  'https://dlp.googleapis.com/v2/infoTypes'
```

The first time that DLP API is used, a 403 response will be returned with a message like the following:
```bash
Cloud Data Loss Prevention (DLP) API has not been used...Enable it by visiting URL
```

After activating, re-run the DLP API call and it will be successful!

# Use the API to de-identify


