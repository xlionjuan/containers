FROM nvcr.io/nvidia/tensorrt:22.04-py3

RUN apt-get update &&\
    apt-get upgrade -y &&\
    apt-get install -y vim libsm6 libxext6 libxrender-dev libgl1-mesa-glx git wget libssl-dev libopencv-dev libspdlog-dev --no-install-recommends &&\
    apt-get clean &&\
    rm -rf /var/lib/apt/lists/*