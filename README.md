# self-host-documint
Guide for self hosting documint

## Overview
Documint comprises mainly 3 services:
- **Documinter**: PDF rendering engine which does not depend on any other service. It works as a REST API that exposes an endpoint that takes a POST request and returns a PDF or image
- **API**: Backend service for documint which depends on `documinter` and a mongodb database passed a enviroment variables. It is a REST API.
- **Web App**: Frontend web application for documnint. Depends on the `API` service only which is passed as an enviroment variable.

**Note** - In production it is important that the web app is served over over https to use reCAPTCHA on login and registration 

## Setup

### 1 Pull images

Setup `gcloud`

```
gcloud auth login
gcloud config set project documint-app
gcloud auth configure-docker us-central1-docker.pkg.dev
```

For setup with service account download service key file and specify [KEY_FILE_PATH] which points to the service key file
```
gcloud auth activate-service-account --key-file=[KEY_FILE_PATH]
gcloud config set project documint-app
gcloud auth configure-docker us-central1-docker.pkg.dev
```

Pull images
```
docker pull us-central1-docker.pkg.dev/documint-app/api-pkg/documint-api:77dd9c5
docker pull us-central1-docker.pkg.dev/documint-app/documinter-pkg/documinter:c7a34d2
docker pull us-central1-docker.pkg.dev/documint-app/web-app-pkg/documint-web-app:77dd9c5
```

> **Note** - Tags will change with each version

### Enviroment variables
API
```
# Required
API_KEY_SECRET = <string>
SESSION_SECRET = <string>

AWS_BUCKET_NAME = <string>
AWS_ACCESS_KEY_ID = <string>
AWS_SECRET_ACCESS_KEY = <string>

CORS_ALLOWED_ORIGINS = <string>
CLIENT_URL = <string>

DB_URL = <string>

DOCUMINT_LICENSE_KEY = <string>

# URL for documinter(required)
DOCUMINT_RENDER_SERVICE_URL = <string>

# Optional for admin accounts
DOCUMINT_ROOT_ACCOUNT_IDS = <string>
DOCUMINT_SOURCE_ACCOUNT_ID = <string>

# Optional for emails such as login confirmations and password resets
SENDGRID_API_KEY = <string>
SENDGRID_FROM_EMAIL = <string>
SENDGRID_USER_LIST_ID = <string>

# if this variable is not present the registration will be allowed captchaless
RE_CAPTCHA_SECRET = <string>

# these are specific to dev enviroments i.e without https
FORCE_HTTPS=false
SESSION_COOKIE_SECURE=false
SESSION_COOKIE_SAME_SITE=lax
```

Web app
```
VITE_API_URL = <string>

# if this variable is not present the registration will be allowed captchaless required if present on API
VITE_RE_CAPTCHA_KEY = <string>

# if this variable is not present the google fonts plugin will be disabled
VITE_GOOGLE_FONTS_API_KEY = <string>
```

### Run
After pulling you can run each image using the docker run command or with docker compose

Example docker run
```
docker run -d -p 8080:80 --name documint-api us-central1-docker.pkg.dev/documint-app/api-pkg/documint-api:alpine
```

Example docker compose
```yaml
version: '3.9'

services:
  # Web Service
  documinter:
    image: us-central1-docker.pkg.dev/documint-app/documinter-pkg/documinter:c7a34d2
    container_name: documinter
    environment:
      - PORT=8080
    ports:
      - "8080:8080" # Expose port 8080 for the web service

  # API Service
  api:
    image: us-central1-docker.pkg.dev/documint-app/api-pkg/documint-api:77dd9c5
    container_name: api
    depends_on:
      - mongodb
      - documinter
    environment:
      - DB_URL=mongodb://mongodb:27017/documintdb?retryWrites=true&w=majority # MongoDB URL
      - DOCUMINT_RENDER_SERVICE_URL=http://documinter:8080/api/1/render # Render service URL
      - PORT=5001
      - API_KEY_SECRET=carbon
      - SESSION_SECRET=DIOXIDE
      - CORS_ALLOWED_ORIGINS=*;http://web_app:3000; http://localhost:3000
      - DOCUMINT_LICENSE_KEY=Y765MjDEEr2xJjzFIJsb # Use a valid license key
      - AWS_BUCKET_NAME=bucket-name 
      - AWS_ACCESS_KEY_ID=id
      - AWS_SECRET_ACCESS_KEY=access-key
      - FORCE_HTTPS=false
      - SESSION_COOKIE_SECURE=false
      - SESSION_COOKIE_SAME_SITE=lax
    ports:
      - "5001:5001" # Expose port 5000 for the API

  # MongoDB Database
  mongodb:
    image: mongo:latest
    container_name: mongodb
    ports:
      - "27017:27017" # Expose MongoDB port
    volumes:
      - mongodb_data:/data/db # Persistent volume for database storage

  # Frontend Web App
  web_app:
    image: us-central1-docker.pkg.dev/documint-app/web-app-pkg/documint-web-app:77dd9c5
    container_name: web_app
    depends_on:
      - api
    environment:
      - VITE_API_URL=http://localhost:5001/1 # API URL
      - PORT=3000
    ports:
      - "3000:3000" # Expose port 3000 for the frontend app

volumes:
  mongodb_data:
```


## Resource allocation and deployment
Currently documint deploys it's services to GCP's [cloud run](https://cloud.google.com/run/docs/overview/what-is-cloud-run). The process is pushing the images to the [artifact registry](https://cloud.google.com/artifact-registry/docs/overview) then selecting the image when deploying a cloud run service, adding the enviroment variables and allocating resources through GCP's online console. These are settings documint is using in production on GCP's cloud run.

### Documinter
- **Memory**: 512mb
- **CPU**: 1
- **Request Timeout**: 300s
- **Maximum Concurrent Instances**: 80
- **Minimum Number Of Instances**: 0
- **Maximum Number Of Instances**: 100

### API
- **Memory**: 512mb
- **CPU**: 1
- **Request Timeout**: 300s
- **Maximum Concurrent Instances**: 80
- **Minimum Number Of Instances**: 0
- **Maximum Number Of Instances**: 100

### Web App
- **Memory**: 512mb
- **CPU**: 1
- **Request Timeout**: 300s
- **Maximum Concurrent Instances**: 80
- **Minimum Number Of Instances**: 0
- **Maximum Number Of Instances**: 100

