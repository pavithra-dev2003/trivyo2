# Project: Spring Boot CI/CD with GitHub Actions and Trivy

This project demonstrates a CI/CD pipeline using GitHub Actions that:

Builds a Spring Boot application with Maven

Creates a Docker image

Scans the image with Trivy for security vulnerabilities

Pushes the image to Docker Hub

This ensures that the application is built, tested for vulnerabilities, and deployed securely in an automated way.

# Features

✅ Automated build using GitHub Actions

✅ Containerized with Docker

✅ Security scanning with Trivy

✅ Deployment-ready Docker image pushed to Docker Hub

# Prerequisites

Before running this workflow, make sure you have:

A Docker Hub account

A GitHub repository with your Spring Boot + Maven project

Secrets configured in your GitHub repo:

DOCKER_USERNAME → Your Docker Hub username

DOCKER_PASSWORD → Docker Hub access token (recommended instead of password)

# CI/CD Workflow

Save this workflow file as:
.github/workflows/docker-ci.yml

name: Build, Scan and Push to Docker Hub

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up JDK 17
        uses: actions/setup-java@v2
        with:
          distribution: 'adopt'
          java-version: '17'

      - name: Build with Maven
        run: mvn clean package
        working-directory: "java-app-demo new"

      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build Docker image
        run: |
          docker build -t ${{ secrets.DOCKER_USERNAME }}/springboot-app:${{ github.run_number }} \
          -f "java-app-demo new/javapack/Dockerfile" "java-app-demo new"

      - name: Install Trivy
        run: |
          sudo apt-get update
          sudo apt-get install -y wget apt-transport-https gnupg lsb-release
          wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
          echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/trivy.list
          sudo apt-get update
          sudo apt-get install -y trivy

      - name: Scan Docker image for CRITICAL vulnerabilities
        run: trivy image --severity CRITICAL ${{ secrets.DOCKER_USERNAME }}/springboot-app:${{ github.run_number }}

      - name: Push Docker image to Docker Hub
        run: docker push ${{ secrets.DOCKER_USERNAME }}/springboot-app:${{ github.run_number }}

      - name: Logout from Docker Hub
        run: docker logout

🛡# Trivy Security Scanning

Trivy
 is an open-source vulnerability scanner for containers and other artifacts.

In this workflow, Trivy scans the Docker image before pushing it to Docker Hub.

Example command:
trivy image --severity CRITICAL username/springboot-app:1


This will scan the image and report only CRITICAL vulnerabilities.
You can extend severity levels:

trivy image --severity HIGH,CRITICAL username/springboot-app:1

# Workflow Steps Explained

Checkout Repository → Pulls your project code from GitHub.

Set up Java 17 → Configures JDK for Maven build.

Build with Maven → Packages your Spring Boot app into a .jar.

Login to Docker Hub → Uses GitHub secrets for secure authentication.

Build Docker Image → Builds the application’s Docker image.

Install Trivy → Installs Trivy vulnerability scanner.

Scan Image → Checks for CRITICAL vulnerabilities.

Push Image → Pushes the secure image to Docker Hub.

Logout → Logs out from Docker Hub to keep pipeline secure.

# Example Output of Trivy Scan

When vulnerabilities are found, logs will show something like:

2025-08-21T12:30:45Z  INFO  Vulnerability scanning started
springboot-app:1 (alpine 3.18)

Total: 5 (CRITICAL: 1, HIGH: 2, MEDIUM: 2, LOW: 0)

┌───────────────┬─────────────────────┬───────────┬──────────────────┐
│ Vulnerability │ Package             │ Severity  │ Installed Version │
├───────────────┼─────────────────────┼───────────┼──────────────────┤
│ CVE-2023-1234 │ openssl             │ CRITICAL  │ 1.1.1l-r7         │
└───────────────┴─────────────────────┴───────────┴──────────────────┘


This ensures you don’t deploy insecure images.

# Status Badge

Add this badge to the top of your README to see workflow status:

![Build, Scan and Push](https://github.com/<YOUR_GITHUB_USERNAME>/<YOUR_REPO_NAME>/actions/workflows/docker-ci.yml/badge.svg)

# Pulling Image from Docker Hub

Once pushed, others can pull your image with:

docker pull <your-docker-username>/springboot-app:<tag>


Example:

docker pull pavithra123/springboot-app:5

# Summary

GitHub Actions automates build → scan → push

Trivy ensures security scanning before pushing images

Docker Hub stores your ready-to-use application images# trivyo2
files are added
