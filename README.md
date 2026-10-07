# sve-image-factory

Builds container images.

This project requires Docker or Podman to bootstrap itself.

The Taskfile provides the task `build:image-builder` to create a container image for [Buildah](https://buildah.io/). This container image also includes the [Amazon ECR Docker Credential Helper](https://github.com/awslabs/amazon-ecr-credential-helper).

Once this image is available to your CI system, you can use it to define CI jobs that build other container images.
