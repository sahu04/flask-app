# GitHub workflow with Trivy and Dockle Container Image Scanning and Report

This GitHub Actions workflow automates the scanning of Docker images using Trivy and Dockle for security vulnerabilities and best practices compliance. Trivy is utilized to identify vulnerabilities in the image's operating system and libraries, while Dockle focuses on Dockerfile best practices.

## Pipeline stages:
### Name-
### The name of your workflow that is displayed on the Github actions page. If you omit this field, it is set to the file name. 
```
name: trivy-scanning
run-name: trivy-dockle-scanning
```
### On -
#### The "on" section specifies the events that trigger the workflow. In this case,it runs when a push event occurs on the main branch or when a pull request is created against the main branch.

```
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  
```
### Jobs
#### A workflow run is made up of one or more jobs. Jobs define the functionality that will be run in the workflow and run in parallel by default.

```
jobs:
  trivy-scanning-job:
    name: trivy-sec-scan
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
```

### Runs-on
#### The runs-on keyword lets you define the OS (Operating System) your workflow should run on, for example, the latest version of ubuntu.

```
runs-on: ubuntu-latest
```
### Steps-

#### A step is an individual task that can run commands in a job. A step can be either an action or a shell command. Each step in a job executes on the same runner, allowing the actions in that job to share data with each other.

```
steps:
      - name: Checkout
        uses: actions/checkout@v3
```
### Build Docker Image
#### This step builds a Docker image from the provided Dockerfile, tagging it with a unique identifier based on the GitHub commit SHA. The resulting image is tagged with the organization name and application name on Docker Hub, ensuring version control and traceability..
```
- name: Build an image from Dockerfile
  run: |
    docker build -t docker.io/organization-name/app-name:${{ github.sha }} .
```
### Trivy Vulnerability Scanning
#### These steps utilize the Trivy GitHub Actions action to perform vulnerability scanning on the specified Docker image. The first step generates a table-format report for critical and high-severity vulnerabilities, while the second step saves a JSON-format report as "trivy-results.json".
```
- name: Run Trivy vulnerability scanner(table-format)
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'docker.io/organization-name/app-name:${{ github.sha }}'
    format: 'table'
    vuln-type: 'os,library'
    severity: 'CRITICAL,HIGH'
```
### dockle security scan
#### This step execute Dockerfile analysis in JSON format for the specified Docker image. It produces a report named "dockle-result.json" and sets the exit code to "0" to ensure successful execution.
```
- name: dockle security scan(json-format)
  uses: goodwithtech/dockle-action@main
  with:
    image: 'docker.io/organization-name/app-name:${{ github.sha }}'
    format: 'json'
    output: 'dockle-result.json'
    exit-code: '0'
```
### Upload report artifact and GitHub Security tab
#### These steps upload the Trivy scan report to GitHub as an artifact named "Trivy-report" and also upload the Trivy scan results in SARIF format to the GitHub Security tab, ensuring visibility and traceability of security findings within the repository.
```
- name: Upload Report
  uses: actions/upload-artifact@v3
  if: always()
  with:
    name: Trivy-report
    path: trivy-results.json
- name: Upload Trivy scan results to GitHub Security tab
  uses: github/codeql-action/upload-sarif@v3
  if: always()
  with:
    sarif_file: 'trivy-results.sarif'
```
