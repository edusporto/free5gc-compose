# FROM gcc:9.4.0 AS builder
# FROM gcc:12.5.0 AS builder
FROM gcc:12-bullseye AS builder

LABEL maintainer="Free5GC <support@free5gc.org>"

ENV DEBIAN_FRONTEND=noninteractive

ARG TARGET_ARCH=x86_64 
# TARGET_ARCH support : x86_64, aarch64(arm64)

# Install dependencies
RUN apt-get update \
    && apt-get install libsctp-dev lksctp-tools iproute2 build-essential -y \
    && wget https://github.com/Kitware/CMake/releases/download/v3.22.1/cmake-3.22.1-linux-${TARGET_ARCH}.sh -O cmake_installer.sh \
    && chmod +x cmake_installer.sh \
    && ./cmake_installer.sh --skip-license \
    && wget https://github.com/aligungr/UERANSIM/archive/refs/tags/v3.2.6.zip -O UERANSIM.zip \
    && unzip UERANSIM.zip \
    && rm UERANSIM.zip \
    && cd ./UERANSIM-3.2.6 \
    && make

# FROM bitnami/minideb:bullseye
FROM gcc:12-bullseye

ENV DEBIAN_FRONTEND=noninteractive

# Install runtime dependencies + ping
RUN apt-get update \
    && apt-get install libsctp-dev lksctp-tools iproute2 iputils-ping procps psmisc build-essential -y \
    && apt-get clean

WORKDIR /ueransim

RUN mkdir -p config/ binder/

COPY --from=builder /UERANSIM/build/nr-gnb .
COPY --from=builder /UERANSIM/build/nr-ue .
COPY --from=builder /UERANSIM/build/nr-cli .
COPY --from=builder /UERANSIM/build/nr-binder binder/
COPY --from=builder /UERANSIM/build/libdevbnd.so binder/

VOLUME [ "/ueransim/config" ]
