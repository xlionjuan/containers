FROM nvcr.io/nvidia/tensorrt:22.04-py3 as dist-packages

FROM scratch

COPY --from=dist-packages /usr/local/lib/python3.8/dist-packages/tensorrt* /