# 🚀 CI/CD Final Project: Tools and Practices

## Project Name: CI/CD Final Project
**Student / Author:** Arya Desavale ([@aryadesavale24-cyber](https://github.com/aryadesavale24-cyber))  
**Course:** Continuous Integration and Continuous Delivery (CI/CD) — IBM DevOps and Software Engineering Professional Certificate  
**Repository:** [https://github.com/aryadesavale24-cyber/ci-cd-final-project](https://github.com/aryadesavale24-cyber/ci-cd-final-project)

---

## Project Overview

This repository contains the completed Final Project demonstrating a full end-to-end Continuous Integration and Continuous Delivery (CI/CD) pipeline.

The solution integrates:
1. **GitHub Actions**: Automated continuous integration running linting checks via `flake8` and unit tests via `nose`.
2. **Tekton & OpenShift Pipelines**: Cloud-native pipeline execution on Red Hat OpenShift, including workspace cleanup, source checkout, code linting, unit testing, container image builds via `buildah`, and deployment to the OpenShift cluster.
3. **Red Hat OpenShift Container Platform**: Deployment of the Python microservice, configuring PersistentVolumeClaims (PVC), and validating application runtime via OpenShift service logs.

---

## Deliverables & Tasks Summary

| Question | Deliverable | Location / Reference | Status |
| :---: | :--- | :--- | :---: |
| **Q1** | GitHub URL of `README.md` with Project Name details | [README.md](https://github.com/aryadesavale24-cyber/ci-cd-final-project/blob/main/README.md) | ✅ Completed |
| **Q2** | GitHub URL of `.github/workflows/workflow.yml` (flake8 lint & nose test) | [.github/workflows/workflow.yml](https://github.com/aryadesavale24-cyber/ci-cd-final-project/blob/main/.github/workflows/workflow.yml) | ✅ Completed |
| **Q3** | GitHub URL of `.tekton/tasks.yml` (cleanup & nose tasks) | [.tekton/tasks.yml](https://github.com/aryadesavale24-cyber/ci-cd-final-project/blob/main/.tekton/tasks.yml) | ✅ Completed |
| **Q4** | Terminal output of successful GitHub Actions workflow (`cicd-github-validate`) | `cicd-github-validate.txt` | ✅ Completed |
| **Q5** | Screenshot of OpenShift PVC details (`oc-pipelines-console-pvc-details.png`) | `screenshots/oc-pipelines-console-pvc-details.png` | ✅ Completed |
| **Q6** | Screenshot of OpenShift Pipeline (`oc-pipelines-oc-final.png`) | `screenshots/oc-pipelines-oc-final.png` | ✅ Completed |
| **Q7** | Screenshot of successful green OpenShift Pipeline (`oc-pipelines-oc-green.png`) | `screenshots/oc-pipelines-oc-green.png` | ✅ Completed |
| **Q8** | Terminal output of OpenShift application logs (`oc-pipeline-application-logs`) | `oc-pipeline-application-logs.txt` | ✅ Completed |

---

## ⚙️ CI Pipeline Configuration (GitHub Actions)

File: `.github/workflows/workflow.yml`

```yaml
name: CI workflow

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    container: python:3.9-slim

    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Lint with flake8
        run: |
          flake8 service --count --select=E9,F63,F7,F82 --show-source --statistics
          flake8 service --count --max-complexity=10 --max-line-length=127 --statistics

      - name: Run unit tests with nose
        run: nosetests -v --with-spec --spec-color --with-coverage --cover-package=app
```

---

## 🛠️ Tekton Pipeline Tasks (OpenShift)

File: `.tekton/tasks.yml`

### 1. Cleanup Task
```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: cleanup
spec:
  description: This task will clean up a workspace by deleting all the files.
  workspaces:
    - name: source
  steps:
    - name: remove
      image: alpine:3
      env:
        - name: WORKSPACE_SOURCE_PATH
          value: $(workspaces.source.path)
      workingDir: $(workspaces.source.path)
      securityContext:
        runAsNonRoot: false
        runAsUser: 0
      script: |
        #!/usr/bin/env sh
        set -eu
        echo "Removing all files from ${WORKSPACE_SOURCE_PATH} ..."
        if [ -d "${WORKSPACE_SOURCE_PATH}" ] ; then
          rm -rf "${WORKSPACE_SOURCE_PATH:?}"/*
          rm -rf "${WORKSPACE_SOURCE_PATH}"/.[!.]*
          rm -rf "${WORKSPACE_SOURCE_PATH}"/..?*
        fi
```

### 2. Nose Unit Tests Task
```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: nose
spec:
  workspaces:
    - name: source
  params:
    - name: args
      description: Arguments to pass to nose
      type: string
      default: "-v"
  steps:
    - name: nosetests
      image: python:3.9-slim
      workingDir: $(workspaces.source.path)
      script: |
        #!/bin/bash
        set -e
        python -m pip install --upgrade pip wheel
        pip install -r requirements.txt
        nosetests $(params.args)
```

---

## 📷 Submission Images

- **PersistentVolumeClaim (PVC)**: `screenshots/oc-pipelines-console-pvc-details.png`
- **Complete Pipeline Pipeline Definition**: `screenshots/oc-pipelines-oc-final.png`
- **Successful Pipeline Run**: `screenshots/oc-pipelines-oc-green.png`
- **Application Logs (`SERVICERUNNING` on port 8000)**: `screenshots/oc-pipelines-app-logs.png`

---

## Author
**Arya Desavale**  
GitHub: [@aryadesavale24-cyber](https://github.com/aryadesavale24-cyber)  
Repository: [aryadesavale24-cyber/ci-cd-final-project](https://github.com/aryadesavale24-cyber/ci-cd-final-project)
