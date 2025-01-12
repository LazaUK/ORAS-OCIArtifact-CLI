# ORAS and Docker CLI: Building Custom Images from OCI Artifacts

This repository demonstrates how to generate **OCI artifact** with **ORAS CLI** client and then build a customised Docker image of an Nginx web service from that artifact. It includes a sample tarball and Dockerfile to facilitate end-to-end testing, from creating an OCI artifact to deploying a fully functional Web site.

> [!NOTE]
> - An [Open Container Initiative (OCI)](https://opencontainers.org/) artifact is a standardised, portable and secure way to package and distribute software.
> - [ORAS](https://oras.land/) enables distribution of artifacts across OCI-compliant registries, like GitHub Container Registry (GHCR).

## Table of Contents
* [Step 1: Create an OCI Artifact](#step-1-create-an-oci-artifact)
* [Step 2: Build a Docker Image](#step-2-build-a-docker-image)
* [Step 3: Deploy a Web site](#step-3-deploy-a-web-site)

## Step 1: Create an OCI Artifact
1. Install ORAS client as described [here](https://oras.land/docs/installation/).
2. Create **Personal Access Token** (PAT) in the GiHub's `Settings -> Developer Settings -> Personal Access Tokens -> Tokens (classic)` section. You would need to allow _read_, _write_ and _delete packages_ permissions in your PAT.
3. Setup environment variables:
``` PowerShell
set GHCR_USER=<GITHUB_USERNAME>
set GHCR_TOKEN=ghp_<GITHUB_TOKEN>
set GHCR_REPO=<GITHUB_REPO>
```
4. Login to GitHub Container Registry (GHCR) with ORAS client:
``` PowerShell
oras login -u %GHCR_USER% -p %GHCR_TOKEN% ghcr.io
```
5. Push tarball and Dockerfile to GHCR as OCI artifact:
``` PowerShell
oras.exe push ghcr.io/%GHCR_USER%/%GHCR_REPO%:v1 demopage.tar.gz:application/x-tar Dockerfile:text/plain
```
6. You can verify that the OCI artifact was created successfully by fetching its manifest file:
``` PowerShell
oras manifest fetch --pretty ghcr.io/%GHCR_USER%/%GHCR_REPO%:v1
```
> [!TIP]
> This step-by-step guide assumes you are using Windows 11 on your development machine. For other operating systems, please consult their documentation on the reference of environment variables.

## Step 2: Build a Docker Image
1. Download the OCI artifact from GHCR:
``` PowerShell
oras pull -o ./download ghcr.io/%GHCR_USER%/%GHCR_REPO%:v1
```
2. Login to GHCR with Docker CLI:
``` PowerShell
docker login ghcr.io -u %GHCR_USER% -p %GHCR_TOKEN%
```
3. Build a Docker image from the downloaded OCI artifact:
``` PowerShell
docker build -t ghcr.io/%GHCR_USER%/%GHCR_REPO%:latest ./download
```
4. Publish newly built Docker image on GHCR:
``` PowerShell
docker push ghcr.io/%GHCR_USER%/%GHCR_REPO%:latest
```

## Step 3: Deploy a Web site
1. Launch a container by pulling the latest version of Docker image from GHCR:
``` PowerShell
docker run -d --name DemoSite -p 8080:80 ghcr.io/%GHCR_USER%/%GHCR_REPO%:latest
```
2. Test Web site by opening http://localhost:8080
