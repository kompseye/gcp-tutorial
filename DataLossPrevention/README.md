# About
The Google Cloud Platform has a service called Data Loss Prevention (DLP). This readme documents key commands to run via the Cloud Shell (browser-based shell available through GCP Console), then using the `gcloud` CLI.

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

# Use the API

## Inspect
Finds potentially sensitive info in ContentItem. Read more [here](https://cloud.google.com/dlp/docs/reference/rest/v2/projects.content/inspect).

API Call:
```bash
# see repo for sample input file: inspect-request.json
curl -s -H 'Content-Type: application/json' -H 'Authorization: Bearer [ACCESS_TOKEN]' 'https://dlp.googleapis.com/v2/projects/my-project-123/content:inspect' -d @inspect-request.json
```

API Response:
```json
{
  "result": {
    "findings": [
      {
        "quote": "(206) 555-0123",
        "infoType": {
          "name": "PHONE_NUMBER"
        },
        "likelihood": "LIKELY",
        "location": {
          "byteRange": {
            "start": "19",
            "end": "33"
          },
          "codepointRange": {
            "start": "19",
            "end": "33"
          }
        },
        "createTime": "2019-06-14T03:45:46.271Z"
      }
    ]
  }
}
```

## Deidentify - Redaction
De-identifies sensitive info from a ContentItem. Read more [here](https://cloud.google.com/dlp/docs/reference/rest/v2/projects.content/deidentify).

API Call:
```bash
# see repo for sample input file: redact-request.json
curl -s -H 'Content-Type: application/json' -H 'Authorization: Bearer [ACCESS_TOKEN]' 'https://dlp.googleapis.com/v2/projects/my-project-123/content:deidentify' -d @redact-request.json
```

API Response:
```json
{
  "item": {
    "value": "My email is [EMAIL_ADDRESS]"
  },
  "overview": {
    "transformedBytes": "16",
    "transformationSummaries": [
      {
        "infoType": {
          "name": "EMAIL_ADDRESS"
        },
        "transformation": {
          "replaceWithInfoTypeConfig": {}
        },
        "results": [
          {
            "count": "1",
            "code": "SUCCESS"
          }
        ],
        "transformedBytes": "16"
      }
    ]
  }
}
```

The email address is redacted: replaced with the special string that represents the [Information Type](https://cloud.google.com/dlp/docs/infotypes-reference). Redaction behavior is made possible because of the requested transformation [replaceWithInfoTypeConfig](https://cloud.google.com/dlp/docs/reference/rest/v2/organizations.deidentifyTemplates#DeidentifyTemplate.ReplaceWithInfoTypeConfig).

deidentity-request.json:
```json
{
  "item": {
     "value":"My email is test@example.com",
   },
   "deidentifyConfig": {
     "infoTypeTransformations":{
          "transformations": [
            {
              "primitiveTransformation": {
                "replaceWithInfoTypeConfig": {}
              }
            }
          ]
        }
    },
    "inspectConfig": {
      "infoTypes": {
        "name": "EMAIL_ADDRESS"
      }
    }
}
```

## Deidentify - Pseudonymization/Tokenization
Sensitive data are replaced with surrogates (also known as tokens) and depending on the transformation, the tokens can be reversed and the data re-identified. Read more [here](https://cloud.google.com/dlp/docs/pseudonymization).

API Call:
```bash
# see repo for sample input file: sample-input-files/tokenization-request.json
# this input data here is structured (tabular) data.
# thus the tokenization-request specifies the transformation as "recordTransformations".
# data could very well be free text, for this use case, see example under "Surrogate annotations"
# https://cloud.google.com/dlp/docs/pseudonymization
curl -s -H 'Content-Type: application/json' -H 'Authorization: Bearer [ACCESS_TOKEN]' 'https://dlp.googleapis.com/v2/projects/my-project-123/content:deidentify' -d @tokenization-request.json
```

API Response:
```json
{
  "item": {
    "table": {
      "headers": [
        {
          "name": "Employee ID"
        },
        {
          "name": "Date"
        },
        {
          "name": "Compensation"
        }
      ],
      "rows": [
        {
          "values": [
            {
              "stringValue": "28777"
            },
            {
              "stringValue": "2015"
            },
            {
              "stringValue": "$10"
            }
          ]
        },
        {
          "values": [
            {
              "stringValue": "28777"
            },
            {
              "stringValue": "2016"
            },
            {
              "stringValue": "$20"
            }
          ]
        },
        {
          "values": [
            {
              "stringValue": "39344"
            },
            {
              "stringValue": "2016"
            },
            {
              "stringValue": "$15"
            }
          ]
        }
      ]
    }
  },
  "overview": {
    "transformedBytes": "15",
    "transformationSummaries": [
      {
        "field": {
          "name": "Employee ID"
        },
        "results": [
          {
            "count": "3",
            "code": "SUCCESS"
          }
        ],
        "fieldTransformations": [
          {
            "fields": [
              {
                "name": "Employee ID"
              }
            ],
            "primitiveTransformation": {
              "cryptoReplaceFfxFpeConfig": {
                "cryptoKey": {
                  "unwrapped": {
                    "key": "YWJjZGVmZ2hpamtsbW5vcA=="
                  }
                },
                "commonAlphabet": "NUMERIC"
              }
            }
          }
        ],
        "transformedBytes": "15"
      }
    ]
  }
}
```

The employee ID has been de-identified: replaced with a token. Note the same token is repeated. This is okay and expected since the same employee ID was in two rows of the input data. The `deidentifyConfig` drives the tokenization behavior; specifically the [cryptoReplaceFfxFpeConfig](https://cloud.google.com/dlp/docs/reference/rest/v2/organizations.deidentifyTemplates#DeidentifyTemplate.CryptoReplaceFfxFpeConfig). The CryptoReplaceFfxFpeConfig replaces an identifier -- specified with the `fields` -- with a surrogate using Format Preserving Encryption.

```json
"deidentifyConfig":{
    "recordTransformations":{
      "fieldTransformations":[
        {
          "primitiveTransformation":{
            "cryptoReplaceFfxFpeConfig":{
              "cryptoKey":{
                "unwrapped":{
                  "key":"YWJjZGVmZ2hpamtsbW5vcA=="
                }
              },
              "commonAlphabet":"NUMERIC"
            }
          },
          "fields":[
            {
              "name":"Employee ID"
            }
          ]
        }
      ]
    }
  }
```

## Reidentify
The deidentified input is based on the cryptoReplaceFfxFpeConfig transformation, thus the tokens can be reversed or reidentified. Read more [here](https://cloud.google.com/dlp/docs/pseudonymization).

API Call:
```bash
# see repo for sample input file: sample-input-files/reidentify-request.json
curl -s -H 'Content-Type: application/json' -H 'Authorization: Bearer [ACCESS_TOKEN]' 'https://dlp.googleapis.com/v2/projects/my-project-123/content:reidentify' -d @reidentify-request.json
```

API Response:
```json
{
  "item": {
    "table": {
      "headers": [
        {
          "name": "Employee ID"
        },
        {
          "name": "Date"
        },
        {
          "name": "Compensation"
        }
      ],
      "rows": [
        {
          "values": [
            {
              "stringValue": "11111"
            },
            {
              "stringValue": "2015"
            },
            {
              "stringValue": "$10"
            }
          ]
        },
        {
          "values": [
            {
              "stringValue": "11111"
            },
            {
              "stringValue": "2016"
            },
            {
              "stringValue": "$20"
            }
          ]
        },
        {
          "values": [
            {
              "stringValue": "22222"
            },
            {
              "stringValue": "2016"
            },
            {
              "stringValue": "$15"
            }
          ]
        }
      ]
    }
  },
  "overview": {
    "transformedBytes": "15",
    "transformationSummaries": [
      {
        "field": {
          "name": "Employee ID"
        },
        "results": [
          {
            "count": "3",
            "code": "SUCCESS"
          }
        ],
        "fieldTransformations": [
          {
            "fields": [
              {
                "name": "Employee ID"
              }
            ],
            "primitiveTransformation": {
              "cryptoReplaceFfxFpeConfig": {
                "cryptoKey": {
                  "unwrapped": {
                    "key": "YWJjZGVmZ2hpamtsbW5vcA=="
                  }
                },
                "commonAlphabet": "NUMERIC"
              }
            }
          }
        ],
        "transformedBytes": "15"
      }
    ]
  }
}

```

The token for the employee ID has been reversed, thus re-identifying the data. The `reidentifyConfig` drives the token reversal behavior.

```json
"reidentifyConfig":{
    "recordTransformations":{
      "fieldTransformations":[
        {
          "primitiveTransformation":{
            "cryptoReplaceFfxFpeConfig":{
              "cryptoKey":{
                "unwrapped":{
                  "key":"YWJjZGVmZ2hpamtsbW5vcA=="
                }
              },
              "commonAlphabet":"NUMERIC"
            }
          },
          "fields":[
            {
              "name":"Employee ID"
            }
          ]
        }
      ]
    }
  }
```