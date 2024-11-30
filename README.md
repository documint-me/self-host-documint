# self-host-documint
Guide for self hosting documint

## Overview
Documint comprises mainly 3 services:
- **Documinter**: PDF rendering engine which does not depend on any other service. It works as a REST API that exposes an endpoint that takes a POST request and returns a PDF or image
- **API**: Backend service for documint which depends on `documinter` and a mongodb database passed a enviroment variables. It is a REST API.
- **Web App**: Frontend web application for documnint. Depends on the `API` service only which is passed as an enviroment variable.

**Note** - In production it is important that the web app is served over over https as it uses reCAPTCHA on login and registration 

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
docker pull [REGION]-docker.pkg.dev/[PROJECT_ID]/[REPOSITORY]/[IMAGE]:[TAG]
docker pull [REGION]-docker.pkg.dev/[PROJECT_ID]/[REPOSITORY]/[IMAGE]:[TAG]
docker pull [REGION]-docker.pkg.dev/[PROJECT_ID]/[REPOSITORY]/[IMAGE]:[TAG]
```

### Enviroment variables

### Run
After pulling you can run using the docker run command or with docker compose

## Resource allocation recommendations
