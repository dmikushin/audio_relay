# Use an official Rust runtime as a parent image
FROM rust:1.82 as builder

RUN apt update && \
    apt install -y --no-install-recommends libdbus-1-dev pkg-config libpulse-dev

# Set CROSS_CONTAINER_IN_CONTAINER to inform `cross` that it is executed from within a container
ENV CROSS_CONTAINER_IN_CONTAINER=true

RUN cargo install cross --git https://github.com/cross-rs/cross

ENV CARGO_HOME=/root/.cargo

# Add Docker's official GPG key:
RUN apt install -y --no-install-recommends ca-certificates curl && \
    install -m 0755 -d /etc/apt/keyrings && \
    curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc && \
    chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
RUN echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    tee /etc/apt/sources.list.d/docker.list > /dev/null && \
    apt update

RUN apt install -y --no-install-recommends docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

RUN apt install -y --no-install-recommends jq
