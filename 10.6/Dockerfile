FROM debian:buster-slim as builder
ARG TARGETARCH

ARG BUILD_DEPENDENCIES="           \
        ca-certificates            \
        devscripts                 \
        software-properties-common \
        wget"

RUN set -ex \
    && apt-get update \
    && apt-get install -y ${BUILD_DEPENDENCIES} \
    && echo "no" | dpkg-reconfigure dash

ARG VERSION=10.6.15

WORKDIR /opt/mariadb

RUN set -ex \
    && wget -q http://downloads.mariadb.org/rest-api/mariadb/${VERSION}/mariadb-${VERSION}.tar.gz \
    && tar -xf mariadb-${VERSION}.tar.gz \
    && rm -f mariadb-${VERSION}.tar.gz \
    && \
    arch="$(dpkg --print-architecture)"; arch="${arch##*-}"; \
    case "$arch" in \
        loongarch64) \
            sed -i 's@^LSBID=.*@LSBID="Debian"@g' mariadb-${VERSION}/debian/autobake-deb.sh; \
            sed -i 's@^LSBVERSION=.*@LSBVERSION="10"@g' mariadb-${VERSION}/debian/autobake-deb.sh; \
            sed -i 's@^LSBNAME=.*@LSBNAME="buster"@g' mariadb-${VERSION}/debian/autobake-deb.sh; \
            ;; \
        *) echo "$arch"; \
            ;; \
    esac;

RUN set -ex \
    && apt-get update \
    && cd mariadb-${VERSION} \
    && AUTOBAKE_PREP_CONTROL_RULES_ONLY=true ./debian/autobake-deb.sh \
    && mk-build-deps -t "apt-get -o Debug::pkgProblemResolver=yes --no-install-recommends -y" -i debian/control

RUN set -ex \
    && cd mariadb-${VERSION} \
    && ./debian/autobake-deb.sh \
    && cd .. \
    && rm -rf mariadb-${VERSION}

FROM debian:buster-slim

WORKDIR /opt/mariadb

COPY --from=builder /opt/mariadb /opt/mariadb/dist

