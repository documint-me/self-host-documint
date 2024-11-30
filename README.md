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
gcloud auth configure-docker
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

DB_CLUSTER = <string>
DB_NAME = <string>
DB_PASSWORD = <string>
DB_USERNAME = <string>

# Alternatively we can use the url format for the database directly
DB_URL = <string>

# Optional
DOCUMINT_ROOT_ACCOUNT_IDS = <string>
DOCUMINT_SOURCE_ACCOUNT_ID = <string>

SENDGRID_API_KEY = <string>
SENDGRID_FROM_EMAIL = <string>
SENDGRID_USER_LIST_ID = <string>

SENTRY_DSN = <string>

# if this variable is not present the registration will be allowed captchaless
RE_CAPTCHA_SECRET = <string>

DOCUMINT_LICENCE_KER = <string>
```

Web app
```
REACT_APP_API_URL = https://staging.api.documint.me/1
REACT_APP_BASE_URL = http://localhost:3000
```



### Run
After pulling you can run each image using the docker run command or with docker compose

Example docker run
```
docker run -d -p 8080:80 --name documinter us-central1-docker.pkg.dev/documint-app/api-pkg/documint-api:alpine
```

Example docker compose
```yaml
version: '3.9'

services:
  # Web Service
  documinter:
    image: documinter us-central1-docker.pkg.dev/documint-app/api-pkg/documint-api:alpine
    container_name: documinter
    ports:
      - "8080:8080" # Expose port 8080 for the web service

  # API Service
  documint_api:
    image: us-central1-docker.pkg.dev/documint-app/api-pkg/documint-api:alpine
    container_name: documint_api
    depends_on:
      - mongodb
      - documinter
    environment:
      - DB_URL=mongodb://mongodb:27017/documintdb # MongoDB URL
      - DOCUMINTER_URL=http://documinter:8080           # Web service URL
    ports:
      - "5000:5000" # Expose port 5000 for the API

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
      - documint_api
    environment:
      - API_URL=http://documint_api:5000 # API URL
    ports:
      - "3000:3000" # Expose port 3000 for the frontend app

volumes:
  mongodb_data:
```



## Resource allocation recommendations
