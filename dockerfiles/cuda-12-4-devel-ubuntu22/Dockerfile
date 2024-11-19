FROM docker.io/nvidia/cuda:12.4.1-cudnn-devel-ubuntu22.04

RUN apt-get update &&\
    DEBIAN_FRONTEND=noninteractive apt-get upgrade -y &&\
    DEBIAN_FRONTEND=noninteractive apt-get install -y vim libsm6 libxext6 libxrender-dev libgl1-mesa-glx git wget libssl-dev libopencv-dev libspdlog-dev --no-install-recommends &&\
    apt-get clean &&\
    rm -rf \
      /root/.cache \
      /var/lib/apt/lists/* \
      /var/tmp/* \
      /tmp/*