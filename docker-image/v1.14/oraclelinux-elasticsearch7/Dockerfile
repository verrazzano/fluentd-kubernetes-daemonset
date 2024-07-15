#
# This Dockerfile was combined with the Dockerfile in github.com/verrazzano/fluentd-docker-image/v1.14/oraclelinux/Dockerfile
#
FROM ghcr.io/oracle/oraclelinux:8-slim as build-base

LABEL maintainer="Verrazzano developers <verrazzano_ww@oracle.com>"
ENV RUBY_VERSION=3.0.7
ENV PATH /usr/local/ruby/bin:/fluentd/vendor/bundle/ruby/$RUBY_VERSION/bin:$PATH
ENV GEM_PATH /fluentd/vendor/bundle/ruby/$RUBY_VERSION
ENV GEM_HOME /fluentd/vendor/bundle/ruby/$RUBY_VERSION
# skip runtime bundler installation
ENV FLUENTD_DISABLE_BUNDLER_INJECTION 1

COPY docker-image/v1.14/oraclelinux-elasticsearch7/Gemfile /fluentd/
COPY docker-image/v1.14/oraclelinux-elasticsearch7/Gemfile.lock /fluentd/
COPY docker-image/v1.14/oraclelinux-elasticsearch7/fluent-plugin-oci-logging-1.0.2.gem /tmp/

RUN microdnf -y update \
    && microdnf install -y --nodocs \
        git-core \
        gcc-c++ \
        patch \
        readline \
        readline-devel \
        libffi-devel \
        openssl-devel \
        make \
        bzip2 \
        autoconf \
        automake \
        libtool \
        bison \
        net-tools \
    && curl -fsSL "https://github.com/rbenv/ruby-build/archive/refs/tags/v20240423.tar.gz" -o /tmp/v20240423.tar.gz \
    && cd /tmp && tar -xzf v20230717.tar.gz \
    && PREFIX=/usr/local /tmp/ruby-build-20240423/install.sh \
    && ruby-build 3.0.7 /usr/local/ruby \
    && gem install oj -N -v 3.10.18 \
    && gem install json -N -v 2.4.1 \
    && gem install async-http -N -v 0.54.0 \
    && gem install fluentd -N -v 1.14.5 \
    && gem install bigdecimal -N -v 1.4.4 \
    && gem install bundler -N --version 2.3.5 \
    && gem install rexml -N -v 3.2.5 \
    && gem install /tmp/fluent-plugin-oci-logging-1.0.2.gem -N \
    && bundle config silence_root_warning true \
    && bundle config set --local path '/fluentd/vendor/bundle' \
    && bundle install --gemfile=/fluentd/Gemfile \
    && gem sources --clear-all \
    && rm -rf /fluentd/vendor/bundle/ruby/3.0.7/cache/* /fluentd/vendor/bundle/ruby/3.0.7/doc/* \
    # for log storage (maybe shared with host)
    && mkdir -p /fluentd/log \
    # configuration/plugins path (default: copied from .)
    && mkdir -p /fluentd/etc /fluentd/plugins \
    && rm -rf /root/cache/*

# Copy configuration files
COPY docker-image/v1.14/oraclelinux-elasticsearch7/conf/fluent.conf /fluentd/etc/
COPY docker-image/v1.14/oraclelinux-elasticsearch7/conf/systemd.conf /fluentd/etc/
COPY docker-image/v1.14/oraclelinux-elasticsearch7/conf/kubernetes.conf /fluentd/etc/
COPY docker-image/v1.14/oraclelinux-elasticsearch7/conf/prometheus.conf /fluentd/etc/
RUN touch /fluentd/etc/disable.conf

# Copy plugins
COPY docker-image/v1.14/oraclelinux-elasticsearch7/plugins /fluentd/plugins/
COPY docker-image/v1.14/oraclelinux-elasticsearch7/entrypoint.sh /fluentd/entrypoint.sh

# COPY LICENSE, README, and SECURITY files to the image
COPY LICENSE README.md SECURITY.md /license/

# Build the final image
FROM ghcr.io/oracle/oraclelinux:8-slim as final

USER root
WORKDIR /home/fluent

RUN microdnf -y update \
    && microdnf clean all \
    && rm -rf /var/cache/yum /var/lib/rpm/* /var/cache/dnf /var/lib/dnf/*

RUN dpkgArch="amd64" \
    tiniVersion="v0.18.0" \
    && curl -fsSL "https://github.com/krallin/tini/releases/download/$tiniVersion/tini-$dpkgArch" -o /usr/local/bin/tini \
    && curl -fsSL "https://github.com/krallin/tini/releases/download/$tiniVersion/tini-$dpkgArch.asc" -o /usr/local/bin/tini.asc \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg --batch --keyserver hkp://keyserver.ubuntu.com --recv-keys 6380DC428747F6C393FEACA59A84159D7001A4E5 \
    && gpg --batch --verify /usr/local/bin/tini.asc /usr/local/bin/tini \
    && rm -r /usr/local/bin/tini.asc \
    && chmod +x /usr/local/bin/tini \
    && groupadd -r fluent && useradd -r -g fluent fluent

COPY --chown=fluent:fluent --from=build-base /fluentd/ /fluentd/
COPY --from=build-base /root/ /root/
COPY --from=build-base /usr/local/ruby/ /usr/local/ruby/
COPY --from=build-base /license/ /license/

# Environment variables
ENV RUBY_VERSION=3.0.7
ENV PATH /usr/local/ruby/bin:/fluentd/vendor/bundle/ruby/$RUBY_VERSION/bin:$PATH
ENV GEM_PATH /fluentd/vendor/bundle/ruby/$RUBY_VERSION
ENV GEM_HOME /fluentd/vendor/bundle/ruby/$RUBY_VERSION
# skip runtime bundler installation
ENV FLUENTD_DISABLE_BUNDLER_INJECTION 1

ENV FLUENTD_CONF="fluent.conf"
ENV FLUENTD_OPT=""

EXPOSE 24224 5140

# ENTRYPOINT to run fluentd as root for /var/log / /var/lib
ENTRYPOINT ["tini", "--", "/fluentd/entrypoint.sh"]
