---
permalink: /wiki/
title: "Wiki"
author_profile: true
redirect_from: 
  - /md/
  - /wiki.html
---

## ROS2 with GUI on MacOS using Docker

Create a docker file with the name "Dockerfile" and the following content
```
FROM ros:jazzy-ros-core
RUN apt-get update && apt-get install -y \
    ros-jazzy-demo-nodes-cpp \
    ros-jazzy-foxglove-bridge \
    ros-jazzy-tf2-ros \
    ros-jazzy-desktop
```
