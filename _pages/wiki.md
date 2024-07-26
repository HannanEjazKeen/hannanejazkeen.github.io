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
FROM ros:humble-ros-core
RUN apt-get update && apt-get install -y \
    ros-humble-demo-nodes-cpp \
    ros-humble-tf2-ros \
    ros-humble-desktop \
    python3-colcon-common-extensions \
    git

RUN mkdir -p ~/ros2_ws/src
WORKDIR "~/ros2_ws"
RUN git clone https://github.com/ros2/examples src/examples -b humble
RUN echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
RUN apt-get install -y python3-rosdep
RUN rosdep init
RUN rosdep update
RUN rosdep install --from-paths src --ignore-src -y --skip-keys "fastcdr rti-connext-dds-6.0.1 urdfdom_headers"
RUN echo "source /usr/share/colcon_cd/function/colcon_cd.sh" >> ~/.bashrc
RUN echo "export _colcon_cd_root=/opt/ros/humble/" >> ~/.bashrc
RUN . /opt/ros/humble/setup.sh
#Following command has two reasons 1) source should be at same command with build else it does not source 2) --executor is sequential because the build crash due to very limited ram (8Gb)
RUN /bin/bash -c "source /opt/ros/humble/setup.bash; MAKEFLAGS='-j1 -l1'; colcon build --executor sequential --symlink-install"
RUN echo "source install/setup.bash" >> ~/.bashrc
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

Inside your docker container, publish a `/chatter` topic:
```
root@154326f5f3gh:/# ros2 run demo_nodes_cpp talker
```

Start a new docker container in another terminal and verify you can see the message being published.
```
docker run --rm -it ros2-image ros2 topic echo /chatter
```

### NoVNC (web application) can be used to work with GUI.

First, we need to create a Docker network that our containers can use to communicate. Let's create a Docker network called ros:
```
docker network create ros
```

We can launch noVNC in a Docker container using the Docker image theasp/novnc:latest. Download the image:
```
docker pull theasp/novnc:latest
```

Then run this image in a new container:
```
docker run -d --rm --net=ros \
   --env="DISPLAY_WIDTH=3000" --env="DISPLAY_HEIGHT=1800" --env="RUN_XTERM=no" \
   --name=novnc -p=8080:8080 \
   theasp/novnc:latest
```

noVNC should now be running as a web application inside the container and listening on port `8080`. Since we have mapped that port to 8080 on the host, we should be able to see the noVNC interface at http://<host name>:8080/vnc.html . For example, if the host is our local machine then that will be `http://localhost:8080/vnc.html` . Open this in a modern web browser (not IE) and click the `Connect` button. You should see a blank desktop.

We can then launch containers that run GUI programs and direct them via the noVNC server to our browser. To do this we need to include the parameters `--net=ros --env="DISPLAY=novnc:0.0"`. Run bash in a new container on the same Docker network and direct its `DISPLAY` to the noVNC server:
```
docker run -it --net=ros --env="DISPLAY=novnc:0.0" --env="ROS_MASTER_URI=http://roscore:11311" ros2-image bash
```

Note that roscore in the last command is the name of the Docker container that we created in the last step.

Then, in the bash shell, to initialize the ROS environment:
```
source ros_entrypoint.sh 
```

Then, for example, to run rviz in the bash shell:
```
ros2 run rviz2 rviz2
```

The rviz UI should appear in the browser.
