---
permalink: /wiki/
title: "Wiki"
author_profile: true
redirect_from: 
  - /md/
  - /wiki.html
---

## ROS2 with GUI on MacOS using Docker

Create a docker file with the name `Dockerfile` and the following content
```
FROM ros:jazzy-ros-core
RUN apt-get update && apt-get install -y \
    ros-jazzy-demo-nodes-cpp \
    ros-jazzy-foxglove-bridge \
    ros-jazzy-tf2-ros \
    ros-jazzy-desktop
```

Go to the dockerfile directory via terminal and build the image with the name `ros2-image`.

```
docker build -t ros2-image .
```

Use the following command to run a docker container
```
docker run --rm -it ros2-image bash
```

The `--rm` flag will remove this container after it exits (to save disk space), and the `-it` option will open an interactive terminal.
