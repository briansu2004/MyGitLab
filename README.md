# MyGitLab

My GitLab

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
