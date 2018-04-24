---
layout: page
title: Docker Images
---

We provide all data we gathered for our paper [_A Graph-based Dataset of Commit History of Real-World Android apps_](/publications#a-graph-based-dataset-of-commit-history-of-real-world-android-apps) in Docker containers to facilitate replication and future research.

The idea is to create a dataset of open-source Android applications which can serve as a base for research. Data on Android apps is spread out over multiple source and finding a large number real-world applications with access to source code requires combining these different databases.

To this end we provide two Docker containers that provide different information on all 8,431 open-source Android apps in this dataset.

For either image to run [Docker needs to be installed.](https://docs.docker.com/install/)

## Graph database

One containers hosts a [Neo4j](https://neo4j.com) graph database which contains metadata of all Android apps in the dataset including metadata of all Git commits.

This Docker image can be pulled directly from [Docker Hub](https://hub.docker.com/r/androidtimemachine/neo4j_open_source_android_apps/):

    docker pull androidtimemachine/neo4j_open_source_android_apps

[Detailed instructions can be found in corresponding documentation.](https://github.com/AndroidTimeMachine/neo4j_open_source_android_apps)

## Git repository snapshots

The second container contains a Gitlab instance with snapshots of all Git repositories in the dataset.

This image containing Git repositories is [built from a fork](https://github.com/af60f75b/gitlab_open_source_android_apps/blob/gitlab_open_source_android_apps/docker/Dockerfile) of the [official Gitlab Docker image.](https://store.docker.com/images/gitlab-enterprise-edition) The main difference is, that instead of in a mountable volume, all Git repositories in out dataset are manually persisted in the image.

With almost 150GB in size the image containing clones of all Git repositories in the dataset exceeds size limits of Docker Hub and needs to be downloaded and loaded manually:

 1. [Download the image (tar archive, 145GB)](https://www.dropbox.com/s/e9ld70s72mxc85y/docker_gitlab_open_source_android_apps.tar.gz?dl=0)
 2. Load the image into Docker: `docker load -i path/to/docker_gitlab_open_source_android_apps.tar.gz`
 3. Use the image as described in the [official Gitlab documentation](https://docs.gitlab.com/omnibus/docker/README.html).

 For instance:

    docker run --rm --detach=true \
        --publish 80:80 --publish 2222:22 \
        --hostname=145.108.225.21 \
        gitlab_open_source_android_apps

The web interface is available at port `80` after Gitlab has started up, which may take several minutes. Log-in is possible with user `root` and password `gitlab`. Git repositories can be cloned from port `2222`.
