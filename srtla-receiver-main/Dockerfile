# Build arguments
ARG SRTLA_BRANCH=master
ARG SLS_TAG=latest

# --- Builder Stage ---
FROM alpine:3.20 AS builder

ARG SRTLA_BRANCH
ENV LD_LIBRARY_PATH=/lib:/usr/lib:/usr/local/lib64
WORKDIR /tmp

# Install deps
RUN apk update && apk add --no-cache \
    linux-headers alpine-sdk cmake tcl openssl-dev zlib-dev spdlog spdlog-dev git

# Clone BELABOX version (Better bonding/performance for Moblin)
RUN git clone -b ${SRTLA_BRANCH} https://github.com/Belabox/srtla.git srtla \
    && cd srtla \
    && git submodule update --init --recursive \
    && cmake -DCMAKE_BUILD_TYPE=Release . \
    && make -j$(nproc)

# --- SLS Stage ---
FROM ghcr.io/openirl/srt-live-server:${SLS_TAG} AS sls-stage

# --- Final Stage ---
FROM alpine:3.20

ENV LD_LIBRARY_PATH=/lib:/usr/lib:/usr/local/lib64

# Keep it clean, added libstdc++ and basic tools
RUN apk update \
    && apk add --no-cache openssl libstdc++ supervisor spdlog \
    && rm -rf /var/cache/apk/*

# Create users
RUN adduser -D -u 3001 -s /bin/sh sls \
    && adduser -D -u 3002 -s /bin/sh srtla

# Copy SRTLA binary
COPY --from=builder /tmp/srtla/srtla_rec /usr/local/bin/

# Copy SLS binaries and libs
COPY --from=sls-stage /usr/local/bin/* /usr/local/bin/
COPY --from=sls-stage /usr/local/lib/libsrt* /usr/local/lib/

# Copy config/scripts
COPY --chmod=755 bin/logprefix /bin/logprefix
COPY conf/sls.conf /etc/sls/sls.conf
COPY conf/supervisord.conf /etc/supervisord.conf

# Expose ports (Standard SRTLA + SLS)
EXPOSE 5000/udp 4000/udp 4001/udp 8080/tcp

CMD ["/usr/bin/supervisord", "--nodaemon", "--configuration", "/etc/supervisord.conf"]