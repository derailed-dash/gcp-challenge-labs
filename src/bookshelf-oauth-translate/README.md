# Bookshelf Application

## Overview

This application allows users to add, list and update books.  It makes use of the following:

- A Python Flask web application (which we run in Gunicorn).
- Flask HTML templates and template inheritance.
- Cloud Firestore, for storing book entries and user profiles.
- GCS for storing book cover images.
- Google Secret Manager, for storing OAuth 2.0 JSON and Flask key.
- Google OAuth 2.0 Consent. This is supported by several endpoints, and URL re-writing.
- Login / Logout / Authorisation.
- Google Cloud Translate API, for converting book descriptions into the users preferred language. (The preferred language is stored in their profile.)

## Setup

Start by cloning this repo to a folder called `bookshelf`.

```bash
export PROJECT='<Your Google Cloud Project ID>'
export REGION='<your region>'

# Create regional Firestore DB
gcloud firestore databases create --location="$REGION" 

# Create regional GCS bucket
gcloud storage buckets create gs://$PROJECT-covers \
  --location="$REGION" --no-public-access-prevention --uniform-bucket-level-access

# Make GCS objects publicly readable
gcloud storage buckets add-iam-policy-binding gs://$PROJECT-covers \
  --member=allUsers --role=roles/storage.legacyObjectReader

# Create venv and activate
python3 -m venv .venv
source .venv/bin/activate

# Install requirements
pip3 install -r ~/bookshelf/requirements.txt
```

## Create OAuth 2.0 Consent

1. APIs & Services --> OAuth Consent Screen
1. External users
1. OAuth Consent screen:
  - App name: Bookshelf
  - User support email: an identity in your project
  - Add authorized domain: cloudshell.dev (which is the domain the application runs on, when running in Cloud Shell)
  - Add address for developer contact.
1. Scopes:
  - Add scopes for openid, auth/userinfo.profile
1. Add test user
1. Create OAuth 2.0 Credentials
  - Application type: Web application
  - Name: Bookshelf
  - Redirect URI: from `echo "https://8080-${WEB_HOST}/oauth2callback"`
1. Download the credential, then upload it to Cloud Shell.

## Store Secrets in Secret Manager

```bash
gcloud services enable secretmanager.googleapis.com

# rename the client secret file
mv ~/client_secret*.json ~/client_secret.json

# Create the client secret in Secret Manager
gcloud secrets create bookshelf-client-secrets --data-file=$HOME/client_secret.json

# Create a random password and store it as a secret called flask-secret-key
# Here we just pipe in 20 chars from from /dev/urandom
# The tr -dc command extracts only matching characters from the generated stream
tr -dc A-Za-z0-9 </dev/urandom | head -c 20 | gcloud secrets create flask-secret-key --data-file=-
```

## Test the app

```bash
cd ~/bookshelf

# This URI should match the host of the redirect URI we created earlier.
# We need to pass this into the application if we're running from Cloud Shell
# otherwise the host will be localhost and the callback will fail.
export EXTERNAL_HOST_URL="https://8080-$WEB_HOST"
~/.venv/bin/gunicorn -b :8080 main:app
```
