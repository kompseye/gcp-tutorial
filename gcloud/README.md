# About
The command-line interaction with the Google Cloud Platform is possible through `gcloud`. This readme documents how to install `gcloud`. The [gcloud reference](https://cloud.google.com/sdk/gcloud/reference/) documents the commands.

# Obtain gcloud with Google Cloud SDK
Install the Google Cloud SDK following [these](https://cloud.google.com/sdk/docs/) instructions, which instruct:
1. Download a GZIP File
1. Extract the contents
1. Configure your path
1. Initialize gcloud with `gcloud init` and following prompts.

# Use gcloud via Docker container
If you use Docker already, an alternative to downloading the Google Cloud SDK to your machine, is to download the [Google Cloud SDK Docker Image](https://github.com/GoogleCloudPlatform/cloud-sdk-docker) and run the `gcloud` CLI via a Docker container.
1. Download Google Cloud SDK Docker image: `docker pull google/cloud-sdk:latest`
1. Verify install by running this command (via a container): `docker run -ti google/cloud-sdk:latest gcloud version`
1. Run `gcloud` (via a container): `docker run -ti google/cloud-sdk:latest /bin/bash`
1. An interative Bash session is opened, where `gcloud` commands can be run
1. Try: `gcloud version`
1. Initialize the configuration: `gcloud auth login`

Do you already service credentials downloaded? The following command utilizes `CLOUDSDK_CONFIG` environment variable which designates a path for GCP configs. The path will actually be a mounted path from the HOST machine. (this means that the contents of the GCP configs will be preserved after the container stops)

1. Run this command
```bash
docker run -ti --rm -e CLOUDSDK_CONFIG=/config/gcp \
                -v /host/machine/gcp/config:/config/gcp \
                -v /host/machine/gcp/certs:/certs  google/cloud-sdk:latest /bin/bash
```
2. Now run this: `gcloud auth login`.
3. Follow the prompts. Google is using OAuth to authorize the Google Cloud SDK to use your Google credentials. After you authorize, a verification code is presented on the browser. Copy that and paste into terminal window which is waiting at prompt `Enter verification code:`
4. List projects: `gcloud projects list`