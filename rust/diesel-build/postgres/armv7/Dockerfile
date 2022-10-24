# syntax=docker/dockerfile:1

# This file was schamelessly stolen from https://github.com/dani-garcia/vaultwarden/tree/main/docker/armv7

# Using multistage build:
# 	https://docs.docker.com/develop/develop-images/multistage-build/
# 	https://whitfin.io/speeding-up-rust-docker-builds/

########################## BUILD IMAGE  ##########################
ARG RUST_VERSION
FROM rust:${RUST_VERSION}-bullseye as build

# Build time options to avoid dpkg warnings and help with reproducible builds.
ENV DEBIAN_FRONTEND=noninteractive \
    LANG=C.UTF-8 \
    TZ=UTC \
    TERM=xterm-256color \
    CARGO_HOME="/root/.cargo" \
    USER="root"


# Create CARGO_HOME folder and don't download rust docs
RUN mkdir -pv "${CARGO_HOME}" \
    && rustup set profile minimal

#
# Install required build libs for armhf architecture.
# hadolint ignore=DL3059
RUN dpkg --add-architecture armhf \
    && apt-get update \
    && apt-get install -y \
        --no-install-recommends \
        libssl-dev:armhf \
        libc6-dev:armhf \
        libpq5:armhf \
        libpq-dev:armhf \
        gcc-arm-linux-gnueabihf \
    #
    # Make sure cargo has the right target config
    && echo '[target.armv7-unknown-linux-gnueabihf]' >> "${CARGO_HOME}/config" \
    && echo 'linker = "arm-linux-gnueabihf-gcc"' >> "${CARGO_HOME}/config" \
    && echo 'rustflags = ["-L/usr/lib/arm-linux-gnueabihf"]' >> "${CARGO_HOME}/config"

# Set arm specific environment values
ENV CC_armv7_unknown_linux_gnueabihf="/usr/bin/arm-linux-gnueabihf-gcc" \
    CROSS_COMPILE="1" \
    OPENSSL_INCLUDE_DIR="/usr/include/arm-linux-gnueabihf" \
    OPENSSL_LIB_DIR="/usr/lib/arm-linux-gnueabihf"


# Creates a dummy project used to grab dependencies
RUN USER=root cargo new --bin /app
WORKDIR /app
