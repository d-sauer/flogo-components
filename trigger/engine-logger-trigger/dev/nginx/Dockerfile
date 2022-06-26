#FROM debian:bookworm-slim as builder
FROM nginx:stable as builder

ENV NGINX_VERSION 1.21.4
ENV OPENTELEMETRY_VERSION v1.1.0

SHELL ["/bin/bash", "-c"]
RUN mkdir -p /install
WORKDIR "/install"

RUN apt-get update \
    && apt-get -y install curl wget git build-essential autoconf libtool pkg-config ca-certificates cmake gcc g++ libcurl4-openssl-dev libpcre3-dev gnupg2 lsb-release apt-transport-https software-properties-common zlib1g-dev \
    && apt-get clean all

#RUN curl -o /etc/apt/trusted.gpg.d/nginx_signing.asc https://nginx.org/keys/nginx_signing.key \
#    && apt-add-repository "deb http://nginx.org/packages/debian `lsb_release -cs` nginx" \
#    && /bin/bash -c 'echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900"' | tee /etc/apt/preferences.d/99nginx
#RUN DEBIAN_FRONTEND=noninteractive TZ="Europe/London" apt-get install --no-install-recommends --no-install-suggests -y nginx

RUN git clone --shallow-submodules --depth 1 --recurse-submodules -b v1.36.4  https://github.com/grpc/grpc   \
    && cd grpc \
    && mkdir -p cmake/build \
    && cd cmake/build \
    && cmake -DgRPC_INSTALL=ON \
             -DgRPC_BUILD_TESTS=OFF \
             -DCMAKE_INSTALL_PREFIX=/install \
             -DCMAKE_BUILD_TYPE=Release \
             -DgRPC_BUILD_GRPC_NODE_PLUGIN=OFF \
             -DgRPC_BUILD_GRPC_OBJECTIVE_C_PLUGIN=OFF \
             -DgRPC_BUILD_GRPC_PHP_PLUGIN=OFF \
             -DgRPC_BUILD_GRPC_PHP_PLUGIN=OFF \
             -DgRPC_BUILD_GRPC_PYTHON_PLUGIN=OFF \
             -DgRPC_BUILD_GRPC_RUBY_PLUGIN=OFF \
              ../.. \
    && make -j2 \
    && make install

RUN git clone --shallow-submodules --depth 1 --recurse-submodules -b v1.3.0 https://github.com/open-telemetry/opentelemetry-cpp.git \
    && cd opentelemetry-cpp \
    && mkdir build \
    && cd build \
    && cmake -DCMAKE_BUILD_TYPE=Release \
             -DCMAKE_INSTALL_PREFIX=/install \
             -DCMAKE_PREFIX_PATH=/install \
             -DWITH_OTLP=ON \
             -DWITH_OTLP_GRPC=ON \
             -DWITH_OTLP_HTTP=OFF \
             -DBUILD_TESTING=OFF \
             -DWITH_EXAMPLES=OFF \
             -DCMAKE_POSITION_INDEPENDENT_CODE=ON \
              .. \
    && make -j2 \
    && make install


RUN mkdir otel-nginx \
    && mkdir -p otel-nginx/build && mkdir -p otel-nginx/src \
    && cd otel-nginx \
    && git clone https://github.com/open-telemetry/opentelemetry-cpp-contrib.git \
    && cd opentelemetry-cpp-contrib/instrumentation/nginx \
    && cp -r src /install/otel-nginx \
    && cp CMakeLists.txt nginx.cmake /install/otel-nginx \
    && cd /install/otel-nginx/build \
    && cmake -DCMAKE_BUILD_TYPE=Release \
             -DCMAKE_PREFIX_PATH=/install \
             -DCMAKE_INSTALL_PREFIX=/usr/share/nginx/modules \
             .. \
    && make -j2 \
    && make install


FROM nginx:stable
# https://github.com/nginxinc/docker-nginx/blob/b18fb328f999b28a7bb6d86e06b0756c1befa21a/stable/debian/Dockerfile

## Setup
RUN mkdir -p /etc/nginx/modules
COPY --from=builder /install/otel-nginx/build/otel_ngx_module.so /etc/nginx/modules/
COPY config/nginx.conf /etc/nginx
COPY config/otel-nginx.toml /etc/nginx


RUN mkdir /etc/nginx/lc
VOLUME ["/etc/nginx/lc"]

STOPSIGNAL SIGQUIT
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
#CMD ["/etc/nginx/nginx", "-g", "daemon off;"]