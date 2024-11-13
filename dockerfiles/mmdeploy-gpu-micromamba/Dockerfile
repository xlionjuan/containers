# bring in the micromamba image so we can copy files from it
FROM registry.hub.docker.com/mambaorg/micromamba:latest as micromamba

FROM nvcr.io/nvidia/tensorrt:22.04-py3

ARG CUDA=11.3
ARG PYTHON_VERSION=3.9
ARG TORCH_VERSION=1.10.0
ARG TORCHVISION_VERSION=0.11.0
ARG ONNXRUNTIME_VERSION=1.15.1
ARG PPLCV_VERSION=0.7.0
ENV FORCE_CUDA="1"
ARG MMCV_VERSION="==2.0.1"
ARG MMENGINE_VERSION=">=0.3.0"

ARG MAMBA_USER=mambauser
ARG MAMBA_USER_ID=57439
ARG MAMBA_USER_GID=57439
ENV MAMBA_USER=$MAMBA_USER
ENV MAMBA_ROOT_PREFIX="/opt/conda"
ENV MAMBA_EXE="/bin/micromamba"

ENV DEBIAN_FRONTEND=noninteractive

### update apt and install libs
RUN apt-get update &&\
    apt-get upgrade -y &&\
    apt-get install -y vim libsm6 libxext6 libxrender-dev libgl1-mesa-glx git wget libssl-dev libopencv-dev libspdlog-dev --no-install-recommends &&\
    apt-get clean &&\
    rm -rf /var/lib/apt/lists/*

COPY --from=micromamba "$MAMBA_EXE" "$MAMBA_EXE"
COPY --from=micromamba /usr/local/bin/_activate_current_env.sh /usr/local/bin/_activate_current_env.sh
COPY --from=micromamba /usr/local/bin/_dockerfile_shell.sh /usr/local/bin/_dockerfile_shell.sh
COPY --from=micromamba /usr/local/bin/_entrypoint.sh /usr/local/bin/_entrypoint.sh
COPY --from=micromamba /usr/local/bin/_dockerfile_initialize_user_accounts.sh /usr/local/bin/_dockerfile_initialize_user_accounts.sh
COPY --from=micromamba /usr/local/bin/_dockerfile_setup_root_prefix.sh /usr/local/bin/_dockerfile_setup_root_prefix.sh


RUN /usr/local/bin/_dockerfile_initialize_user_accounts.sh && \
    /usr/local/bin/_dockerfile_setup_root_prefix.sh

#USER $MAMBA_USER

SHELL ["/usr/local/bin/_dockerfile_shell.sh"]

### install miniconda pytorch openmim + mmcv onnxruntime
RUN /bin/micromamba install -y python=${PYTHON_VERSION} conda-build pyyaml numpy ipython cython typing typing_extensions mkl mkl-include ninja -c defaults && \
    /bin/micromamba install pytorch==${TORCH_VERSION} torchvision==${TORCHVISION_VERSION} cudatoolkit=${CUDA} -c pytorch -c conda-forge -y &&\
    /opt/conda/bin/pip install --no-cache-dir openmim watchdog &&\
    /bin/micromamba clean -ya &&\
    /opt/conda/bin/mim install --no-cache-dir "mmcv"${MMCV_VERSION} onnxruntime-gpu==${ONNXRUNTIME_VERSION} mmengine${MMENGINE_VERSION} && \
    /opt/conda/bin/pip install mmdet --no-cache-dir

# ### install miniconda pytorch openmim + mmcv onnxruntime
# RUN curl -fsSL -v -o ~/miniconda.sh -O  https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh  && \
#     chmod +x ~/miniconda.sh && \
#     bash ~/miniconda.sh -b -p /opt/conda && \
#     rm ~/miniconda.sh && \
#     /opt/conda/bin/conda install -y python=${PYTHON_VERSION} conda-build pyyaml numpy ipython cython typing typing_extensions mkl mkl-include ninja && \
#     /opt/conda/bin/conda install pytorch==${TORCH_VERSION} torchvision==${TORCHVISION_VERSION} cudatoolkit=${CUDA} -c pytorch -c conda-forge -y &&\
#     /opt/conda/bin/pip install --no-cache-dir openmim watchdog &&\
#     /opt/conda/bin/conda clean -ya &&\
#     /opt/conda/bin/mim install --no-cache-dir "mmcv"${MMCV_VERSION} onnxruntime-gpu==${ONNXRUNTIME_VERSION} mmengine${MMENGINE_VERSION}

ENV PATH=/opt/conda/bin:$PATH
WORKDIR /root/workspace

### cp trt from pip to conda
#RUN cp -r /usr/local/lib/python${PYTHON_VERSION}/dist-packages/tensorrt* /opt/conda/lib/python${PYTHON_VERSION}/site-packages/
RUN cp -r /usr/local/lib/python3.8/dist-packages/tensorrt* /opt/conda/lib/python${PYTHON_VERSION}/site-packages/

### build sdk
### install mmdeploy
### get onnxruntime
ENV ONNXRUNTIME_DIR=/root/workspace/onnxruntime-linux-x64-${ONNXRUNTIME_VERSION}
ENV TENSORRT_DIR=/workspace/tensorrt
#ARG VERSION

ENV BACKUP_LD_LIBRARY_PATH=$LD_LIBRARY_PATH
ENV LD_LIBRARY_PATH=/usr/local/cuda/compat/lib.real/:$LD_LIBRARY_PATH

RUN wget https://github.com/microsoft/onnxruntime/releases/download/v${ONNXRUNTIME_VERSION}/onnxruntime-linux-x64-${ONNXRUNTIME_VERSION}.tgz &&\
    tar -zxvf onnxruntime-linux-x64-${ONNXRUNTIME_VERSION}.tgz &&\
    rm onnxruntime-linux-x64-${ONNXRUNTIME_VERSION}.tgz &&\

    git clone -b 1.x --depth 1 https://github.com/open-mmlab/mmdeploy &&\
    cd mmdeploy &&\
    if [ -z ${VERSION} ] ; then echo "No MMDeploy version passed in, building on 1.x" ; else git checkout tags/v${VERSION} -b tag_v${VERSION} ; fi &&\
    git submodule update --init --recursive --depth 1 &&\
    mkdir -p build &&\
    cd build &&\
    cmake -DMMDEPLOY_TARGET_BACKENDS="ort;trt" .. &&\
    make -j$(nproc) &&\
    cd .. &&\
    /opt/conda/bin/mim install -e . &&\

    cd /root/workspace &&\
    git clone --branch v${PPLCV_VERSION} --depth 1 https://github.com/openppl-public/ppl.cv.git &&\
    cd ppl.cv &&\
    ./build.sh cuda&&\

    cd /root/workspace/mmdeploy &&\
    rm -rf build/CM* build/cmake-install.cmake build/Makefile build/csrc &&\
    mkdir -p build && cd build &&\
    cmake .. \
        -DMMDEPLOY_BUILD_SDK=ON \
        -DMMDEPLOY_BUILD_EXAMPLES=ON \
        -DCMAKE_CXX_COMPILER=g++ \
        -Dpplcv_DIR=/root/workspace/ppl.cv/cuda-build/install/lib/cmake/ppl \
        -DTENSORRT_DIR=${TENSORRT_DIR} \
        -DONNXRUNTIME_DIR=${ONNXRUNTIME_DIR} \
        -DMMDEPLOY_BUILD_SDK_PYTHON_API=ON \
        -DMMDEPLOY_TARGET_DEVICES="cuda;cpu" \
        -DMMDEPLOY_TARGET_BACKENDS="ort;trt" \
        -DMMDEPLOY_CODEBASES=all &&\
    make -j$(nproc) && make install &&\
    export SPDLOG_LEVEL=warn &&\
    if [ -z ${VERSION} ] ; then echo "Built MMDeploy dev-1.x for GPU devices successfully!" ; else echo "Built MMDeploy version v${VERSION} for GPU devices successfully!" ; fi &&\
    
    rm -rf /root/workspace/ppl.cv &&\
    rm -rf /root/workspace/mmdeploy &&\
    rm -rf ${ONNXRUNTIME_DIR}

ENV LD_LIBRARY_PATH="/root/workspace/mmdeploy/build/lib:${BACKUP_LD_LIBRARY_PATH}"

