#AI Platform Samples
https://github.com/GoogleCloudPlatform/ai-platform-samples

## ai-platform-samples/notebooks/notebooks-ci-showcase

### APIs aktivieren
e.g. BigQuery
https://console.cloud.google.com/flows/enableapi?apiid=bigquery&hl=de&_ga=2.12295284.-1331973218.1565080239


cd /home/danny/OneDrive/DataScience/41_DannyJRa.github.io/90_Tensorflow/ai-platform-samples/notebooks/notebooks-ci-showcase/

bash run_test.sh

worked with CPU

## Copy result notebobook ucket to folder test
gsutil cp -r gs://dl-platform-temp3/notebook-ci-showcase test

## Limitations

So far no artefacts

## GCP Authorization
https://cloud.google.com/sdk/docs/authorizing?hl=de

bq show --encryption_service_account

Using service account
Blog_Telegram_Picture
https://www.freecodecamp.org/news/how-to-build-a-bot-to-automate-your-mindless-tasks-using-python-and-google-bigquery-a34faf7fb74/

via Json file

export GOOGLE_APPLICATION_CREDENTIALS='[/home/danny/OneDrive/DataScience/02_CloudComputing/00_Secrets/kubeflowtest-bigquery.json]'

export GOOGLE_APPLICATION_CREDENTIALS="/home/danny/OneDrive/DataScience/02_CloudComputing/00_Secrets/kubeflowtest-bigquery.json"
echo $GOOGLE_APPLICATION_CREDENTIALS


Manually setting the context credentials:

from google.cloud.bigquery import magics
from google.oauth2 import service_account
credentials = (service_account.Credentials.from_service_account_file('/home/danny/OneDrive/DataScience/02_CloudComputing/00_Secrets/kubeflowtest-bigquery.json'))
magics.context.credentials = credentials


gcloud config set project kubeflowtest-248212

gcloud services list --enabled



# Parameters via papermill


microk8s.stop
