FROM turlucode/ros-noetic:cuda11.4.2

MAINTAINER Athanasios Tasoglou <athanasios@tasoglou.net>
LABEL Description="ROS-Noetic-Desktop with CUDA 11.4.2 and cuDNN 8 support (Ubuntu 20.04)" Vendor="TurluCode" Version="1.0"

# change source to aliyun
RUN sed -i s/archive.ubuntu.com/mirrors.aliyun.com/g /etc/apt/sources.list \
    && sed -i s/security.ubuntu.com/mirrors.aliyun.com/g /etc/apt/sources.list


# Install packages without prompting the user to answer any questions
ENV DEBIAN_FRONTEND noninteractive 

# CUDNN Runtime-packages
ENV NV_CUDNN_VERSION 8.2.4.15
ENV NV_CUDNN_PACKAGE "libcudnn8=$NV_CUDNN_VERSION-1+cuda11.4"
LABEL com.turlucode.ros.cudnn="${NV_CUDNN_VERSION}"

RUN apt-get update && apt-get install -y --no-install-recommends \
    ${NV_CUDNN_PACKAGE} && \
    rm -rf /var/lib/apt/lists/*


# CUDNN Devel-packages
ENV NV_CUDNN_PACKAGE_DEV "libcudnn8-dev=$NV_CUDNN_VERSION-1+cuda11.4"

RUN apt-get update && apt-get install -y --no-install-recommends \
    tmux \
    gedit \
    vim \
    mlocate \
    ros-noetic-ros-controllers \
    ros-noetic-roboticsgroup-upatras-gazebo-plugins  \
    ${NV_CUDNN_PACKAGE} \
    ${NV_CUDNN_PACKAGE_DEV} && \
    rm -rf /var/lib/apt/lists/*

RUN apt-mark hold libcudnn8

# install miniconda for root
ENV PATH="/root/miniconda3/bin:${PATH}"
ARG PATH="/root/miniconda3/bin:${PATH}"

RUN wget \
    https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh \
    && mkdir /root/.conda \
    && bash Miniconda3-latest-Linux-x86_64.sh -b \
    && rm -f Miniconda3-latest-Linux-x86_64.sh \
# init conda for shell
RUN conda init bash \
    && conda init zsh

# Make RUN commands use the new environment:
RUN echo "conda activate ros_base" >> ~/.bashrc \
    && echo "conda activate ros_base" >> ~/.zshrc \
    && conda --version

COPY ./env/ros_base.yml .
RUN conda env create -f ros_base.yml
# Initialize conda in bash config fiiles:
SHELL ["conda", "run", "-n", "ros_base", "/bin/bash", "-c"]

# add catkin_pkg to path for using in conda env
ENV PYTHONPATH "${PYTHONPATH}:/usr/lib/python3/dist-packages"
# uninstall the pip or else there will be error when using pip command in container
RUN apt-get remove --purge -y python3-pip
# uninstall the older version setuptools installed by the ros noetic
RUN pip uninstall -y setuptools

RUN git clone https://github.com/stevengj/nlopt.git && cd nlopt && mkdir build && cd build \
    && cmake .. && make && sudo make install \
    && rm -rf cmake

# Launch terminator
CMD ["terminator"]
