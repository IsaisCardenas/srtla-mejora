# Build arguments
ARG SRTLA_BRANCH=main
ARG SLS_TAG=latest

# --- Builder Stage ---
FROM alpine:3.20 AS builder
ARG SRTLA_BRANCH
WORKDIR /tmp

RUN apk update && apk add --no-cache \
    linux-headers alpine-sdk cmake tcl openssl-dev zlib-dev spdlog spdlog-dev git

# Usamos la repo de OpenIRL que ya conocemos
RUN git clone -b ${SRTLA_BRANCH} https://github.com/OpenIRL/srtla.git srtla \
    && cd srtla \
    && cmake -DCMAKE_BUILD_TYPE=Release . \
    && make -j$(nproc)

# --- Final Stage ---
FROM alpine:3.20
ENV LD_LIBRARY_PATH=/lib:/usr/lib:/usr/local/lib64

RUN apk update && apk add --no-cache openssl libstdc++ supervisor spdlog && rm -rf /var/cache/apk/*

COPY --from=builder /tmp/srtla/srtla_rec /usr/local/bin/
# Aquí traemos el resto de componentes (SLS)
COPY --from=ghcr.io/openirl/srt-live-server:latest /usr/local/bin/* /usr/local/bin/
COPY --from=ghcr.io/openirl/srt-live-server:latest /usr/local/lib/libsrt* /usr/local/lib/

COPY --chmod=755 bin/logprefix /bin/logprefix
COPY conf/sls.conf /etc/sls/sls.conf
COPY conf/supervisord.conf /etc/supervisord.conf

EXPOSE 5000/udp 4001/udp
CMD ["/usr/bin/supervisord", "--nodaemon", "--configuration", "/etc/supervisord.conf"]