# Stage 1: build the toolchain
FROM ubuntu:20.04 as toolchain-build

# Source: https://github.com/GomSpace/avr32-toolchain/blob/av32-toolchain-3.4.2/scripts/Dockerfile
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    git \
    bison \
    flex \
    libncurses-dev \
    wget \
    libmpfr-dev \
    libgmp-dev \
    build-essential

WORKDIR /root

# Build and Install old version of texinfo
RUN wget http://ftp.gnu.org/gnu/texinfo/texinfo-4.13.tar.gz && tar xf texinfo-4.13.tar.gz && cd texinfo-4.13 && ./configure && make && make install

# Clone the toolchain repo
RUN mkdir /root/avr32-toolchain
WORKDIR /root/avr32-toolchain

COPY . /root/avr32-toolchain

# Clone the tools
RUN ./scripts/avr32-clone-all.sh

WORKDIR /root/avr32-toolchain/scripts

# Build the tools
RUN ./build-all.sh

# Stage 2: build the devcontainer
FROM ubuntu:20.04 as devcontainer

# Install needed software
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    python3 \
    python3-pip \
    python-is-python3 \
    git \
    sudo \
    unzip \
    wget \
    build-essential \
    protobuf-compiler \
    doxygen \
    graphviz \
    gnupg2 \
    screen \
    minicom

# Copy the toolchain
COPY --from=toolchain-build /root/avr32-toolchain/gs-avr32-toolchain-3.4.2_gs1 /root/toolchain

# Download protoc-gen-buff
WORKDIR /root
RUN wget https://github.com/pseudomuto/protoc-gen-doc/releases/download/v1.5.1/protoc-gen-doc_1.5.1_linux_amd64.tar.gz \
    && tar xvf protoc-gen-doc_1.5.1_linux_amd64.tar.gz \
    && rm protoc-gen-doc_1.5.1_linux_amd64.tar.gz

RUN cp protoc-gen-doc /usr/local/bin

# Install the toolchain
WORKDIR /root/toolchain
RUN ./install-avr32.sh

# Create the user
ARG USERNAME=user
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Create the user
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME -s /bin/bash \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME

USER $USERNAME

# Update the PATH
ENV PATH "$PATH:/usr/local/avr32/bin"