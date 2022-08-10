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
