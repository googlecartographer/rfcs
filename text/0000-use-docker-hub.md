# Push Docker Images to Docker Hub
## Summary
[summary]: #summary

Add continuous deployment of Docker images built by the CI pipelines to a public repository on Docker Hub.
These images could then be re-used by dependent repositories as the base image,  which could make Docker builds more efficient.

## Motivation
[motivation]: #motivation

* Currently, Docker-based CI builds of Cartographer projects are slow and inefficient
  * Caching only speeds up the build of a repository if nothing has changed in the dependency's master (cache invalidation and rebuilding the dependencies otherwise)
  * long build times block pull requests, Travis build occasionally cancel with a timeout error, etc...
* facilitates integration in downstream projects

## Approach
[approach]: #approach

Add a `googlecartographer` organization on Docker Hub, to which the latest built Docker images of the master branches are pushed, e.g. `googlecartographer/cartographer` or `googlecartographer/cartographer_ros`.

For continuous integration, we push `googlecartographer/cartographer_base` images that contain pre-installed 3rd party dependencies like protobuf, Ceres...
The version of these dependencies is fixed in the build scripts (directory `scripts/`).

### Pushing Images in Travis CI

See the Travis documentation for how to set up credentials: [Pushing a Docker Image to a Registry](https://docs.travis-ci.com/user/docker/#pushing-a-docker-image-to-a-registry)

We have to make sure that we only push Docker images built from the master branch:

```bash
#!/bin/bash

if [[ $TRAVIS_BRANCH == "master" ]] && [[ $TRAVIS_PULL_REQUEST == "false" ]]; then
    echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
    docker push googlecartographer/$DOCKER_IMAGE_NAME
fi
```

`$DOCKER_IMAGE_NAME` would be `cartographer:xenial` or `cartographer_ros:melodic` for example.

### Pulling Images

Use the pushed images from Docker Hub as base images for dependent repositories instead of re-building everything from scratch.

E.g. in `cartographer_ros/Dockerfile.kinetic`:

```
FROM googlecartographer/cartographer_base:xenial
FROM ros:kinetic
```
to make re-building the base dependencies in `cartographer_ros` unnecessary in builds triggered by pull requests or on the master branch.
Multiple `FROM` statements are supported by Docker and duplicate layers are efficiently handled (ToDo: test that locally with Cartographer).

## Discussion Points
[discussion]: #discussion

So far, this is a rough idea.
Any concerns or suggestions?
