# GitLab workflow with Trivy and Dockle Container Image Scanning and Report
This GitLab CI/CD pipeline automates the build process for container images and performs security scans using Trivy and Dockle.

## Prerequisites
- GitLab CI/CD configured with appropriate permissions for accessing repositories and executing pipelines.
- Access to the GitLab Container Registry for pushing and pulling container images.

## Pipeline Stages
### variable:
These GitLab CI/CD configuration variables and services enable Docker-in-Docker (DinD) functionality, facilitating container image building and manipulation within the pipeline. The variables define Docker host details and image registry paths, while the service specification ensures proper DinD setup with appropriate entrypoint and commands.
```
variables:
  DOCKER_HOST: tcp://docker:2375/
  DOCKER_DRIVER: overlay2
  DOCKER_TLS_CERTDIR: ""
  IMAGE_TAG: $CI_COMMIT_SHORT_SHA
  IMAGE_REGISTRY: $CI_REGISTRY_IMAGE:$IMAGE_TAG

services:
  - name: docker:dind
    entrypoint: ["env", "-u", "DOCKER_HOST"]
    command: ["dockerd-entrypoint.sh"]
```
### build:
 This GitLab CI/CD job the script block, it builds a Docker image tagged with the specified image registry, logs into the GitLab Container Registry using provided credentials, and finally pushes the built image to the registry for further use.
- and the values of this `CI_REGISTRY_USER` and `CI_REGISTRY_PASSWORD` needs to set up in (gitlab-settings-ci/cd-variables)
```
build_image:
  stage: build
  image: docker:stable
  script:
    - docker build -t $IMAGE_REGISTRY .
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker push $IMAGE_REGISTRY
```
### test
The trivy_scan job in this GitLab CI/CD pipeline is integral to the security assessment process.It fetches the Docker image, runs Trivy scans in JSON and table formats, saving the findings in trivy-report.json. This thorough process pinpoints vulnerabilities, aiding security improvements. With allow_failure: true, it ensures smooth pipeline flow, even if Trivy faces problems, seamlessly integrating into the CI/CD workflow.
```
before_script:
    - |
      export TRIVY_VERSION=$(wget -qO - "https://api.github.com/repos/aquasecurity/trivy/releases/latest" | grep '"tag_name":' | sed -E 's/.*"v([^"]+)".*/\1/')
    - echo $TRIVY_VERSION
    - wget --no-verbose https://github.com/aquasecurity/trivy/releases/download/v${TRIVY_VERSION}/trivy_${TRIVY_VERSION}_Linux-64bit.tar.gz -O - | tar -zxvf -
  allow_failure: true
  script:
    - docker pull $IMAGE_REGISTRY
    - docker images
    - ./trivy image --scanners vuln --format json -o trivy-report.json $IMAGE
    - ./trivy image --scanners vuln --format table $IMAGE
     artifacts:
    reports:
      container_scanning: trivy-report.json
```
### scan
The dockle_scan job in this GitLab CI/CD pipeline performs Docker image scanning using Dockle. It fetches the Docker image, runs Dockle to analyze image security, and saves results in dockle-registry.json. This ensures Dockerfile best practices adherence and enhances image security. The job's artifacts include the generated JSON report, aiding in post-scan analysis and continuous improvement efforts.
```
 before_script:
    - wget https://github.com/goodwithtech/dockle/releases/download/v0.4.13/dockle_0.4.13_Linux-64bit.tar.gz
    - tar -xzf dockle_0.4.13_Linux-64bit.tar.gz
    - mv dockle /usr/local/bin/
    - export PATH=$PATH:/usr/local/bin  # Add the directory containing Dockle binary to the system path
    - dockle --version
  script:
    - docker pull $IMAGE_REGISTRY # Pull the image from the registry if it's not already available
    - docker images
    - /usr/local/bin/dockle -f json -o dockle-registry.json --exit-code 1 --exit-level fatal $IMAGE
  artifacts:
    paths:
      - dockle-registry.json
```
