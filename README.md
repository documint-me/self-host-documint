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
docker pull us-central1-docker.pkg.dev/documint-app/api-pkg/documint-api:alpine
docker pull us-central1-docker.pkg.dev/documint-app/documinter-pkg/documinter:sharp
docker pull us-central1-docker.pkg.dev/documint-app/web-app-pkg/documint-web-app:alpine
```

**Note** - Tags will change with each version

### Enviroment variables
API
```
AUTH_API_KEY_SECRET = <string>
SESSION_SECRET = <string>

AWS_S3_BUCKET_NAME = <string>
AWS_ACCESS_KEY_ID = <string>
AWS_SECRET_ACCESS_KEY = <string>
AWS_DEFAULT_REGION = <string>
AWS_S3_ENDPOINT = <string>

CORS_ALLOWED_ORIGINS = <string>

DB_URL = <string>

# URL for documinter
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

DOCUMINT_LICENSE_KEY = <string>
```

Web app
```
REACT_APP_API_URL = <string>
REACT_APP_BASE_URL = <string>

# if this variable is not present the registration will be allowed captchaless required if present on API
REACT_APP_RE_CAPTCHA_KEY = <string>
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
    image: us-central1-docker.pkg.dev/documint-app/documinter-pkg/documinter:sharp
    container_name: documinter
    environment:
      - PORT=8080
    ports:
      - "8080:8080" # Expose port 8080 for the web service

  # API Service
  api:
    image: us-central1-docker.pkg.dev/documint-app/api-pkg/documint-api:alpine
    container_name: api
    depends_on:
      - mongodb
      - documinter
    environment:
      - DB_URL=mongodb://mongodb:27017/documintdb # MongoDB URL
      - DOCUMINT_RENDER_SERVICE_URL=http://documinter:8080 # Render service URL
      - PORT=5001
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
    image: us-central1-docker.pkg.dev/documint-app/web-app-pkg/documint-web-app:alpine
    container_name: web_app
    depends_on:
      - api
    environment:
      - REACT_APP_API_URL=http://api:5000 # API URL
      - REACT_APP_BASE_UR=http://web_app:3000
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

