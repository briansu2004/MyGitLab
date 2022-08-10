# MyGitLab

My GitLab

## GitLab 2FA

Got a tricky issue for a while and finally resolved it

Root cause:

The timezone needs to be the same with the GitLab server!

Steps:

```dos
- install Google auth app in my iPhone
- Scan QR code to add my GitLab account
- General -> Settings -> Time zone -> if my TZ is different with then GL server, don't use auto, change it to the same as GL server's (very import)
- Hooray
```

## GitLab CI/CD pipeline

### Simple

.gitlab-ci.yml

```yaml
stages:
    - build
    - test

build:
    stage: build
    script:
        - echo "Building"
        - mkdir build
        - touch build/info.txt
    artifacts:
        paths:
            - build/

test:
    stage: test
    script:
        - echo "Testing"
        - test -f "build/info.txt"
```

### GitLab Runner

### Docker Image

```yaml
...
```

### Free GitLab + Free Netlify ???

### GitLab + GCP (Google Cloud SDK) + Skaffold

Skaffold is a command-line tool developed by Google that makes it really easy to setup CI/CD for local, development and production environments with Kubernetes. It handles the full workflow of building and pushing docker containers to the docker container registry, and deploying applications to Kubernetes.

Skaffold doesn't require any server-side components to be installed in the Kubernetes cluster. All it needs is to have the necessary credentials to access the Kubernetes control plane API server. This is great because it means that you can run Skaffold pretty much from anywhere.

Dockerfile_skaffold

```docker
FROM google/cloud-sdk:alpine

ENV DOCKER_HOST tcp://localhost:2375/
ENV DOCKER_DRIVER overlay

RUN apk upgrade --no-cache \
  # Install common tools
  && apk add --no-cache bash curl wget make git py-pip \
  gcc g++ linux-headers binutils-gold gnupg libstdc++ libgcc ca-certificates tar jq \
  # Install skaffold
  && curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64 \
  && mv skaffold /usr/local/bin/ \
  && chmod +x /usr/local/bin/skaffold \
  && skaffold version
```

Docker build

```bash
docker build -f Dockerfile_skaffold -t <my-username>/skaffold-and-gcloud 
```

Docker push

```bash
docker login
docker push <my_docker_repo>/skaffold-and-gcloud:latest
```

.gitlab-ci.yml

```yaml
image: "<my_docker_repo>/skaffold-and-gcloud"

services:
 - docker:dind

stages:
  - build
  - review
  - production

variables:
  DOMAIN_NAME: example.com
  PRODUCTION_DOMAIN_NAME: example.com

build_review:
  stage: build
  environment:
    name: $CI_COMMIT_REF_NAME
    url: "https://$CI_ENVIRONMENT_SLUG.$DOMAIN_NAME"
    on_stop: stop_review
  script:
    - <do_build>
  only:
    - branches
  except:
    - main

build_production:
  stage: build
  environment:
    name: production
    url: "https://$PRODUCTION_DOMAIN_NAME"
  script:
    - echo 'doing build_production'
  only:
    - main

recreate_review:
  stage: review
  script:
    - echo 'doing recreate_review'
  environment:
    name: $CI_COMMIT_REF_NAME
    url: "https://$CI_ENVIRONMENT_SLUG.$DOMAIN_NAME"
    on_stop: stop_review
  when: manual
  only:
    - branches
  except:
    - main
  dependencies:
  - build_review

deploy_review:
  stage: review
  script:
    - echo 'doing deploy_review'
  environment:
    name: $CI_COMMIT_REF_NAME
    url: "https://$CI_ENVIRONMENT_SLUG.$DOMAIN_NAME"
    on_stop: stop_review
  when: manual
  only:
    - branches
  except:
    - main
  dependencies:
  - build_review

stop_review:
  stage: review
  script:
    - echo 'doing stop_review'
  environment:
    name: $CI_COMMIT_REF_NAME
    url: "https://$CI_ENVIRONMENT_SLUG.$DOMAIN_NAME"
    action: stop
  dependencies:
    - build_review
  when: manual
  except:
    - main

deploy_production:
  stage: production
  script:
    - echo doing deploy_production
  environment:
    name: production
    url: "https://$PRODUCTION_DOMAIN_NAME"
  when: manual
  only:
    - main
  dependencies:
  - build_production
```

dind : docker in docker

### GitLab + GCP

.gitlab-ci.yml

```yaml
default:
    image: google/cloud-sdk:alpine
    before_script:
        - gcloud config set project <my_gcp_project>
        - gcloud auth activate-service-account --key-file $GCP_SERVICE_CREDS

build:
    stage: build
    script:
        - gcloud build submit --tag gcr.io/<my_image_url>:latest

deploy:
    stage: deploy
    script:
        - gcloud run deploy <my_cloud_run_service> --image gcr.io/<my_image_url>:latest --platform managed --region us-west1 --allow-unauthenticated
```

### How to trigger a GitLab pipeline

#### Use a CI/CD job - yes

For example, to trigger a pipeline on the `main` branch of `project-B` when a tag is created in `project-A`, add the following job to project A's `.gitlab-ci.yml` file:

```yaml
trigger_pipeline:
  stage: deploy
  script:
    - 'curl --fail --request POST --form token=$MY_TRIGGER_TOKEN --form ref=main "https://gitlab.example.com/api/v4/projects/123456/trigger/pipeline"'
  rules:
    - if: $CI_COMMIT_TAG
```

In this example:

```dos
123456 is the project ID for project-B. The project ID is displayed at the top of every project's landing page. 
(can it be changed as )

The rules cause the job to run every time a tag is added to project-A.

MY_TRIGGER_TOKEN is a masked CI/CD variables that contains the trigger token.
```

#### Use a webhook - maybe

```dos
https://gitlab.example.com/api/v4/projects/<project_id>/ref/<ref_name>/trigger/pipeline?token=<token>
```

#### Use cURL - nah

```dos
curl --request POST \
     --form token=<token> \
     --form ref=<ref_name> \
     "https://gitlab.example.com/api/v4/projects/<project_id>/trigger/pipeline"
```

or

```dos
curl --request POST \
    "https://gitlab.example.com/api/v4/projects/<project_id>/trigger/pipeline?token=<token>&ref=<ref_name>"
```
