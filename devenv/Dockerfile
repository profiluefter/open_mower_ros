FROM osrf/ros:noetic-desktop-full AS base
ARG MY_UID
ARG MY_GID
ARG MY_USER

# Create a new group with provided name and ID
RUN groupadd -g ${MY_GID} -o ${MY_USER}

# Create a new user, assign it to the group, and set its shell to bash
RUN useradd -m -u ${MY_UID} -g ${MY_GID} -o -s /bin/bash ${MY_USER} && yes ${MY_USER} | passwd ${MY_USER}

RUN echo $MY_USER ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$MY_USER \
    && chmod 0440 /etc/sudoers.d/$MY_USER

RUN apt-get update && apt-get install -y zsh git sudo ssh gdb rsync

RUN ( \
    echo 'LogLevel DEBUG2'; \
    echo 'PermitRootLogin yes'; \
    echo 'PasswordAuthentication yes'; \
    echo 'ListenAddress 0.0.0.0:2222'; \
    echo 'Subsystem sftp /usr/lib/openssh/sftp-server'; \
  ) > /etc/ssh/sshd_clion \
  && mkdir /run/sshd
RUN bash -c "ssh-keygen -A"

RUN echo "$MY_USER ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# Ensure rosdep is initialized and refreshed during image build.
RUN if [ ! -f /etc/ros/rosdep/sources.list.d/20-default.list ]; then rosdep init; fi \
    && rosdep update --rosdistro $ROS_DISTRO

RUN grep -qxF "source /opt/ros/noetic/setup.bash" "/home/${MY_USER}/.bashrc" \
    || echo "source /opt/ros/noetic/setup.bash" >> "/home/${MY_USER}/.bashrc"

# Create a new image layer based on the base image for storing dependencies.
# Only package manifests are copied here so source code changes do not bust this cache layer.
FROM base AS dependencies

ENV DEBIAN_FRONTEND=noninteractive

WORKDIR /tmp/open_mower_ros

COPY --link src/mower_comms_v1/package.xml src/mower_comms_v1/package.xml
COPY --link src/mower_comms_v2/package.xml src/mower_comms_v2/package.xml
COPY --link src/mower_logic/package.xml src/mower_logic/package.xml
COPY --link src/mower_map/package.xml src/mower_map/package.xml
COPY --link src/mower_msgs/package.xml src/mower_msgs/package.xml
COPY --link src/mower_simulation/package.xml src/mower_simulation/package.xml
COPY --link src/mower_utils/package.xml src/mower_utils/package.xml
COPY --link src/open_mower/package.xml src/open_mower/package.xml
COPY --link src/lib/ftc_local_planner/package.xml src/lib/ftc_local_planner/package.xml
COPY --link src/lib/ntrip_client/package.xml src/lib/ntrip_client/package.xml
COPY --link src/lib/slic3r_coverage_planner/package.xml src/lib/slic3r_coverage_planner/package.xml
COPY --link src/lib/xbot_driver_gps/package.xml src/lib/xbot_driver_gps/package.xml
COPY --link src/lib/xbot_framework/package.xml src/lib/xbot_framework/package.xml
COPY --link src/lib/xbot_monitoring/package.xml src/lib/xbot_monitoring/package.xml
COPY --link src/lib/xbot_msgs/package.xml src/lib/xbot_msgs/package.xml
COPY --link src/lib/xbot_positioning/package.xml src/lib/xbot_positioning/package.xml
COPY --link src/lib/xbot_remote/package.xml src/lib/xbot_remote/package.xml
COPY --link src/lib/xbot_rpc/package.xml src/lib/xbot_rpc/package.xml
COPY --link src/lib/xesc_ros/xesc/package.xml src/lib/xesc_ros/xesc/package.xml
COPY --link src/lib/xesc_ros/xesc_2040_driver/package.xml src/lib/xesc_ros/xesc_2040_driver/package.xml
COPY --link src/lib/xesc_ros/xesc_driver/package.xml src/lib/xesc_ros/xesc_driver/package.xml
COPY --link src/lib/xesc_ros/xesc_interface/package.xml src/lib/xesc_ros/xesc_interface/package.xml
COPY --link src/lib/xesc_ros/xesc_msgs/package.xml src/lib/xesc_ros/xesc_msgs/package.xml
COPY --link src/lib/xesc_ros/xesc_yfr4_driver/package.xml src/lib/xesc_ros/xesc_yfr4_driver/package.xml
COPY --link src/lib/xesc_ros/vesc_driver/package.xml src/lib/xesc_ros/vesc_driver/package.xml

RUN apt-get update && \
    rosdep install --from-paths . --ignore-src --default-yes --rosdistro $ROS_DISTRO && \
    rm -rf /var/lib/apt/lists/*

# Create another image layer for the development environment
FROM dependencies AS devenv


CMD ["sudo", "/usr/sbin/sshd", "-D", "-e", "-f", "/etc/ssh/sshd_clion"]
