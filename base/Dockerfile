ARG BUILD_FROM=ghcr.io/hassio-addons/base:19.0.0
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Environment variables
ENV PATH="/usr/local/bin:$PATH"

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Install base system
ARG PYTHON_VERSION="v3.14.2"
ARG PYTHON_SHA256="2299dae542d395ce3883aca00d3c910307cd68e0b2f7336098c8e7b7eee9f3e9"
ARG PYTHON_PIP_VERSION="25.3"
ARG PYTHON_SETUPTOOLS_VERSION="80.10.2"

# hadolint ignore=DL3003,DL4006,SC2155
RUN \
    apk add --no-cache --virtual .build-dependencies \
        bzip2-dev=1.0.8-r6 \
        dpkg-dev=1.22.15-r0 \
        dpkg=1.22.15-r0 \
        expat-dev=2.7.3-r0 \
        findutils=4.10.0-r0 \
        gcc=14.2.0-r6 \
        gdbm-dev=1.24-r0 \
        libffi-dev=3.4.8-r0 \
        libnsl-dev=2.0.1-r1 \
        libtirpc-dev=1.3.5-r0 \
        linux-headers=6.14.2-r0 \
        make=4.4.1-r3 \
        musl-dev=1.2.5-r10 \
        ncurses-dev=6.5_p20250503-r0 \
        openssl-dev=3.5.4-r0 \
        pax-utils=1.3.8-r1 \
        readline-dev=8.2.13-r1 \
        sqlite-dev=3.49.2-r1 \
        tar=1.35-r3 \
        tcl-dev=8.6.16-r0 \
        tk-dev=8.6.16-r0 \
        tk=8.6.16-r0 \
        util-linux-dev=2.41-r9 \
        xz-dev=5.8.1-r0 \
        xz=5.8.1-r0 \
        zlib-dev=1.3.1-r2 \
        zstd-dev=1.5.7-r0 \
    \
    && curl -J -L -o /tmp/python.tar.xz \
        "https://www.python.org/ftp/python/${PYTHON_VERSION#v}/Python-${PYTHON_VERSION#v}.tar.xz" \
    \
    && echo "$PYTHON_SHA256 /tmp/python.tar.xz" | sha256sum -c - \
    \
    && mkdir -p /usr/src/python \
    && tar -xJC /usr/src/python --strip-components=1 -f /tmp/python.tar.xz \
    && cd /usr/src/python \
    \
    && gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
    && ./configure \
        --build="$gnuArch" \
        --enable-loadable-sqlite-extensions \
        --enable-optimizations \
        --enable-option-checking=fatal \
        --enable-shared \
        --with-lto \
        --with-system-expat \
        --without-ensurepip \
    \
    && make -j "$(nproc)" \
        EXTRA_CFLAGS="-DTHREAD_STACK_SIZE=0x100000" \
        LDFLAGS="-Wl,--strip-all" \
        PROFILE_TASK="" \
        EXTRA_CFLAGS="-fno-omit-frame-pointer -mno-omit-leaf-frame-pointer" \
    && make install \
    \
    && find /usr/local \
        -type f \
        -executable \
        -not \( -name '*tkinter*' \) \
        -exec scanelf \
        --needed \
        --nobanner \
        --format '%n#p' '{}' ';' \
            | tr ',' '\n' \
            | sort -u \
            | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
            | xargs -rt apk add --no-cache --virtual .python-rundeps \
    \
    && cd /usr/local/bin \
    && ln -s idle3 idle \
    && ln -s pydoc3 pydoc \
    && ln -s python3 python \
    && ln -s python3-config python-config \
    \
    && curl -J -L -o /tmp/get-pip.py \
        'https://bootstrap.pypa.io/get-pip.py' \
    \
    && python /tmp/get-pip.py \
        --disable-pip-version-check \
        --no-cache-dir \
        --no-compile \
        "pip==$PYTHON_PIP_VERSION" \
        "setuptools==$PYTHON_SETUPTOOLS_VERSION" \
    \
    && apk del --no-cache --purge .build-dependencies \
    && rm -f -r \
        /usr/src \
        /tmp/* \
    \
    && python3 --version \
    && pip3 --version \
    \
    && find /usr -depth \
        \( \
            \( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
            -o \
            \( -type f -a \( -name '*.pyc' -o -name '*.pyo' -o -name 'libpython*.a' \) \) \
        \) -exec rm -rf '{}' +

# Entrypoint & CMD
ENTRYPOINT ["/init"]

# Build arugments
ARG BUILD_DATE
ARG BUILD_REF
ARG BUILD_VERSION
ARG BUILD_REPOSITORY

# Labels
LABEL \
    io.hass.name="Addon Python base for ${BUILD_ARCH}" \
    io.hass.description="Home Assistant Community Add-on: ${BUILD_ARCH} Python base image" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="base" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.title="Addon Python base for ${BUILD_ARCH}" \
    org.opencontainers.image.description="Home Assistant Community Add-on: ${BUILD_ARCH} Python base image" \
    org.opencontainers.image.vendor="Home Assistant Community Add-ons" \
    org.opencontainers.image.authors="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://addons.community" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/master/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
