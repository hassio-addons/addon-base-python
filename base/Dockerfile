ARG BUILD_FROM=hassioaddons/base:3.0.1
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Environment variables
ENV \
    PATH="/usr/local/bin:$PATH" \
    GPG_KEY="0D96DF4D4110E5C43FBFB17F2D347EA6AA65421D" \
    PYTHON_VERSION="3.7.3" \
    PYTHON_PIP_VERSION="19.0.3"

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Install base system
RUN \
    apk add --no-cache --virtual .build-dependencies \
        coreutils \
        dpkg-dev dpkg \
        expat-dev \
        findutils \
        gcc \
        gdbm-dev \
        libc-dev \
        libffi-dev \
        libnsl-dev \
        libtirpc-dev \
        linux-headers \
        make \
        ncurses-dev \
        openssl-dev \
        pax-utils \
        readline-dev \
        sqlite-dev \
        tcl-dev \
        tk \
        tk-dev \
        util-linux-dev \
        xz-dev \
        zlib-dev \
        bzip2-dev \
        gnupg \
        tar=1.32-r0 \
        xz \
    \
    && curl -J -L -o /tmp/python.tar.xz \
        "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" \
    && curl -J -L -o /tmp/python.tar.xz.asc \
        "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc" \
    \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg \
        --batch \
        --keyserver ha.pool.sks-keyservers.net \
        --recv-keys "$GPG_KEY" \
    && gpg \
        --batch \
        --verify /tmp/python.tar.xz.asc /tmp/python.tar.xz \
    && { command -v gpgconf > /dev/null && gpgconf --kill all || :; } \
    \
    && mkdir -p /usr/src/python \
    && tar -xJC /usr/src/python --strip-components=1 -f /tmp/python.tar.xz \
    && cd /usr/src/python \
    \
    && gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
    && ./configure \
        --build="$gnuArch" \
        --enable-loadable-sqlite-extensions \
        --enable-shared \
        --with-system-expat \
        --with-system-ffi \
        --without-ensurepip \
    \
    && make -j "$(nproc)" \
        EXTRA_CFLAGS="-DTHREAD_STACK_SIZE=0x100000" \
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
    && find /usr/local -depth \
        \( \
            \( -type d -a \( -name test -o -name tests \) \) \
            -o \
            \( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
        \) -exec rm -rf '{}' + \
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
        "pip==$PYTHON_PIP_VERSION" \
    \
    && find /usr/local -depth \
        \( \
            \( -type d -a \( -name test -o -name tests \) \) \
            -o \
            \( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
        \) -exec rm -rf '{}' +  \
    \
    && apk del --purge .build-dependencies \
    && rm -f -r \
        /usr/src \
        "$GNUPGHOME" \
        /tmp/* \
    \
    && python3 --version \
    && pip3 --version

# Entrypoint & CMD
ENTRYPOINT ["/init"]

# Build arugments
ARG BUILD_DATE
ARG BUILD_REF
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="base" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Franck Nijhof <frenck@addons.community>" \
    org.label-schema.description="Community Hass.io Add-ons: ${BUILD_ARCH} Python Base image" \
    org.label-schema.build-date=${BUILD_DATE} \
    org.label-schema.name="Addon Python base for ${BUILD_ARCH}" \
    org.label-schema.schema-version="1.0" \
    org.label-schema.url="https://addons.community" \
    org.label-schema.usage="https://github.com/hassio-addons/addon-base-python/blob/master/README.md" \
    org.label-schema.vcs-ref=${REF} \
    org.label-schema.vcs-url="https://github.com/hassio-addons/addon-base-python" \
    org.label-schema.vendor="Franck Nijhof"
