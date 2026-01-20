# Build arguments
ARG SRTLA_BRANCH=main
ARG SLS_TAG=latest

# --- Builder Stage ---
FROM alpine:3.20 AS builder
ARG SRTLA_BRANCH
WORKDIR /tmp

# Instalamos dependencias básicas
RUN apk update && apk add --no-cache \
    linux-headers alpine-sdk cmake tcl openssl-dev zlib-dev spdlog spdlog-dev git

# CLONAR Y ACTUALIZAR SUBMODULOS (Esto arregla el error de argparse)
RUN git clone -b ${SRTLA_BRANCH} https://github.com/OpenIRL/srtla.git srtla \
    && cd srtla \
    && git submodule update --init --recursive \
    && cmake -DCMAKE_BUILD_TYPE=Release . \
    && make -j$(nproc)

# --- Final Stage ---
FROM alpine:3.20
ENV LD_LIBRARY_PATH=/lib:/usr/lib:/usr/local/lib64

RUN apk update && apk add --no-cache openssl libstdc++ supervisor spdlog && rm -rf /var/cache/apk/*

# Traemos el binario compilado
COPY --from=builder /tmp/srtla/srtla_rec /usr/local/bin/

# Traemos los componentes de SLS
COPY --from=ghcr.io/openirl/srt-live-server:latest /usr/local/bin/* /usr/local/bin/
COPY --from=ghcr.io/openirl/srt-live-server:latest /usr/local/lib/libsrt* /usr/local/lib/

# Configuraciones
COPY --chmod=755 bin/logprefix /bin/logprefix
COPY conf/sls.conf /etc/sls/sls.conf
COPY conf/supervisord.conf /etc/supervisord.conf

EXPOSE 5000/udp 4001/udp
CMD ["/usr/bin/supervisord", "--nodaemon", "--configuration", "/etc/supervisord.conf"]