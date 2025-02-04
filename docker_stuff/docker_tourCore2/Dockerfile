# Dockerfile that is supposed to replace the one in tour-guide-robot to build the image to test fcclip in visual-language-navigation
# The dockerfile builds Ubuntu 24.04 with ROS2 Jazzy and CUDA 12.4

ARG base_img
FROM $base_img
LABEL maintainer="Leonardo Gori"

ARG CUDA_VERSION=12.1

# # Non-interactive installation mode
ENV DEBIAN_FRONTEND=noninteractive

# Iterate over all the Ubuntu versions to find the latest CUDA version, starting from the most recent
RUN for ubuntu_v in 2404 2204 2004 1804 1604 1404 1304 1204; do \
        echo "Looking for CUDA version "$CUDA_VERSION" in Ubuntu $ubuntu_v"; \
        echo "Looking into: https://developer.download.nvidia.com/compute/cuda/repos/ubuntu$ubuntu_v/x86_64/"; \
        # extract cuda major and minor versions from the full version string
        CUDA_MAJOR=$(echo "$CUDA_VERSION" | cut -d. -f1) && CUDA_MINOR=$(echo "$CUDA_VERSION" | cut -d. -f2) && \
        # get the latest version of the specified major and minor versions
        CUDA_FILE=$(curl -s https://developer.download.nvidia.com/compute/cuda/repos/ubuntu$ubuntu_v/x86_64/ | grep -oP "(?<=href=')[^']+" | grep "^cuda-$CUDA_MAJOR-$CUDA_MINOR" | tail -1); \
        if [ -n "$CUDA_FILE" ]; then \
            echo "Found CUDA file: $CUDA_FILE"; \
            # get the latest version of the keyring
            CUDA_KEYRING=$(curl -s https://developer.download.nvidia.com/compute/cuda/repos/ubuntu$ubuntu_v/x86_64/ | grep -oP "(?<=href=')[^']+" | grep "^cuda-keyring" | tail -1); \
            # download the keyring
            wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu$ubuntu_v/x86_64/$CUDA_KEYRING; \
            break; \
        fi; \
    done; \
    # check if the CUDA version was found, otherwise exit
    if [ -z "$CUDA_FILE" ]; then \
        echo "CUDA version not found"; \
        exit 1; \
    fi

# RUN wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb
RUN sudo dpkg -i cuda-keyring*.deb
RUN sudo apt-get update
# RUN sudo apt-get upgrade -y
RUN CUDA_MAJOR=$(echo "$CUDA_VERSION" | cut -d. -f1) && CUDA_MINOR=$(echo "$CUDA_VERSION" | cut -d. -f2) && \
    sudo apt-get install -y cuda-$CUDA_MAJOR-$CUDA_MINOR

RUN echo "export PATH=/usr/local/cuda-$CUDA_VERSION/bin${PATH:+:${PATH}}" >> ~/.bashrc
RUN echo "export LD_LIBRARY_PATH=/usr/local/cuda-$CUDA_VERSION/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}" >> ~/.bashrc
RUN . ~/.bashrc

RUN rm cuda*