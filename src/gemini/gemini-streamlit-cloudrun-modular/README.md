```bash
export PROJECT='<Your Google Cloud Project ID>'
export REGION='<your region>'

# Enable APIs
gcloud services enable \
  cloudbuild.googleapis.com \
  cloudfunctions.googleapis.com \
  run.googleapis.com \
  logging.googleapis.com \
  storage-component.googleapis.com \
  aiplatform.googleapis.com

python3 -m venv gemini-streamlit
source gemini-streamlit/bin/activate
pip install -r requirements.txt
```

To run the application locally:

```bash
streamlit run app.py \
  --browser.serverAddress=localhost \
  --server.enableCORS=false \
  --server.enableXsrfProtection=false \
  --server.port 8080
```

To upload to Google Artifact Registry:

```bash
export AR_REPO=chef-repo
export SERVICE_NAME=chef-streamlit-app

# Create a GAR repo
gcloud artifacts repositories create "$AR_REPO" \
  --location="$REGION" --repository-format=Docker

# Allow authentication to the repo
gcloud auth configure-docker "$GCP_REGION-docker.pkg.dev"

# Build to GAR - this takes a few minutes
gcloud builds submit \
  --tag "$REGION-docker.pkg.dev/$PROJECT/$AR_REPO/$SERVICE_NAME"
```

To deploy to Cloud Run:

```bash
# Deploy to Cloud Run - this takes a couple of minutes
gcloud run deploy "$SERVICE_NAME" \
  --port=8080 \
  --image="$REGION-docker.pkg.dev/$PROJECT/$AR_REPO/$SERVICE_NAME" \
  --allow-unauthenticated \
  --region=$REGION \
  --platform=managed  \
  --project=$PROJECT \
  --set-env-vars=GCP_PROJECT=$PROJECT,GCP_REGION=$REGION
  ```