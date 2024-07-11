# Secure-Container-Image-Building Image for Data Science (SCIBIDS)
A base image called "SCIBIDS" (pronounced as "skee-beeds") to build container images with minimal (or zero) vulnerabilities.

Think of SCIBIDS as your handy toolbox to build container images, the main difference between SCIBIDS and other container images is that SCIBIDS is only used for building container images and should never be used to run a container.

SCIBIDS build dependencies are left out in the final stage of image-building which reduces attack surface and enhances security. Think of it this way - you dont need your heavy spanner after tightening your nuts and bolts.

## Description
Repository contains a `Dockerfile` used to build `scibids`, which can be stored on your machine for personal use. It is then used in multi-stage builds to build python packages for the containerized applications.

Refer to `dockerfile-sample` folder for reference on how to use `scibids`.

## Features
SCIBIDS is an Alpine-based image that can build common data science python packages out of the box such as `pandas` (which uses `pyarrow` under the hood - PITA to build from scratch) and `numpy`.

The reason for using Alpine linux is to take advantage of available lightweight and secure Alpine-based images to unload our built python packages, which we then keep as our final image.

Feel free to add on system dependencies inside SCIBIDS' `Dockerfile` to build specific python packages.

## Dependencies
Application source code needs to be written in python 3.12, with all python packages compatible with this version.

## How to use `scibids` in a multi-stage build

1. Use the first `FROM` statement to use `python:3.12-alpine3.20` to import application source code (e.g `requirements.txt`, etc.).
```dockerfile
# Base Image
FROM python:3.12-alpine3.20 AS base

# Set build arguments
ARG BASE_PATH=/base

# Copy requirements.txt
COPY requirements.txt ${BASE_PATH}/
```
2. Use the second `FROM` statement to use `scibids` to build the python packages, using requirements from the first stage.
```dockerfile
# Use SCIBIDS to build python packages
FROM scibi:latest as build-base

# Set build arguments
ARG BASE_PATH=/base
ARG PYTHON_DEP_BUILD_PATH=/dockerbuild_python_deps

...

# Copy requirements.txt
COPY --from=base ${BASE_PATH}/requirements.txt ./

# Install Python packages
RUN pip3 install --no-deps -r requirements.txt --target "${PYTHON_DEP_BUILD_PATH}"

```
3. Use the last `FROM` statement to use `python:3.12-alpine3.20` to copy the python packages from the second stage and install specific system dependencies used by the python packages.
```dockerfile
# Production Environment
FROM python:3.12-alpine3.20

...

# Install system dependencies for python packages
RUN apk add --no-cache \
    curl-dev \
    gcc \
    gcompat \
    libcurl \
    libstdc++ \
    libthrift \
    re2-dev

# Copy python dependencies
COPY --from=build-base ${PYTHON_DEP_BUILD_PATH} ${TASK_ROOT}/
```
