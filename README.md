# Multi-Stage DevOps CI Pipeline (Application Repository)

This repository contains a containerized Python Flask web application integrated with a automated Jenkins declarative CI pipeline. It serves as the **Continuous Integration (CI)** driver for a local GitOps lifecycle.

## Project Architecture & Workflow
1. **Code Push:** Developer pushes code modifications to this repository.
2. **Automated Build:** Local Jenkins detects the change, parses the `Jenkinsfile`, and triggers a multi-stage execution framework.
3. **Containerization:** Jenkins builds a lean Docker container image using a multi-stage layer strategy from the `Dockerfile`.
4. **Registry Storage:** The built image is securely pushed and tagged using the Jenkins credential manager to a target public registry.
5. **GitOps Trigger:** Jenkins checks out the sister infrastructure repository (`k8s-manifest-repo`), dynamically overwrites the deployment version tag using stream editing (`sed`), and pushes the updated configuration back to GitHub.

---

## Repository File Structure
* `app.py`: High-performance Python lightweight Flask web backend.
* `Dockerfile`: Container image build instructions based on a minimized, secure Linux base layers footprint.
* `Jenkinsfile`: Fully dynamic automation orchestration file utilizing decoupled runtime credentials injection.

---

## Tech Stack
* **Language Backend:** Python 3.9 (Flask framework)
* **Automation Automation:** Jenkins Pipeline Engine
* **Containerization Engine:** Docker Desktop (WSL 2 Backend Engine)
