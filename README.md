# Yocto Pipline

This repository briefly describes to process of building platform specific Yocto based images for the [sorrir](https://sorrir.io/) IoT testbed. All images are based on Yocto Warrior

## Overview

* Yocto builds take place in a CI/CD system (GitLab Pipelines)
* Custom layers are imported for use-case specific needs
* Mender is used to deploy and update them
* k3s is used as a kubernetes container orchestrator

### Yocto

This folder contains a Dockerfile used to build a Yocto build environment. Later on, this is used by a build runner in GitLab, but could be easily applied to GitHub Actions or any other CI/CD build tool.

Within the Dockerfile, the following steps are executed:

1. Install the Yocto Build environment and dependencies based on [the official documentation](https://www.yoctoproject.org/docs/1.8/yocto-project-qs/yocto-project-qs.html#ubuntu)
2. Google [Repo](https://gerrit.googlesource.com/git-repo/) is installed since Mender makes heavy use of it
3. A unprivileged user is created and configured that later runs the actual build

### Image example

This folder contains an example on how to build a Raspberry Pi image for our testbed.

Hereby the following process is executed:

1. Prepare the working directory
2. Download the appropriate mender base configuration
3. Download our custom layers
4. Source the build environment
5. Configure the layers with some debug output
6. Build the image
7. Pass the built images to the next stage to upload it to a running mender instance

### Mender

This folder contains a Mender deployment based on their [initial release](https://mender.io/blog/official-helm-chart-for-mender-2-5). This deployment makes use of [kustomize](https://github.com/kubernetes-sigs/kustomize)

The following process is executed:

* Manually run `helm repo add mender https://charts.mender.io`
* Manually run `kustomize build ./mender/mender --enable-helm . | kubectl apply -f -`
* Minio is installed
* MongoDB is installed
* Mender is installed

After a first login to the mender webinterface you can generate the necessarry credentials used in the second pipeline step of our "Image example" above.

## Deployment

Based on this example, you can now create your initial deployment by duming the CI artifact `sdcard` image to your device. Any subsequent update can be automatically deployed using mender.
