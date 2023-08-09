FROM ghcr.io/oracle/oraclelinux:8-slim

LABEL maintainer="Verrazzano developers <verrazzano_ww@oracle.com>"
USER root
WORKDIR /home/fluent
ENV TINI_VERSION=0.18.0
ENV RUBY_VERSION=3.0.6
ENV PATH /usr/local/ruby/bin:/fluentd/vendor/bundle/ruby/3.0.6/bin:$PATH
ENV GEM_PATH /fluentd/vendor/bundle/ruby/3.0.6
ENV GEM_HOME /fluentd/vendor/bundle/ruby/3.0.6
# skip runtime bundler installation
ENV FLUENTD_DISABLE_BUNDLER_INJECTION 1

COPY docker-image/v1.14/oraclelinux-elasticsearch7/Gemfile /fluentd/
COPY docker-image/v1.14/oraclelinux-elasticsearch7/Gemfile.lock /fluentd/
COPY docker-image/v1.14/oraclelinux-elasticsearch7/fluent-plugin-oci-logging-1.0.2.gem /tmp/
#RUN buildDeps="make gcc-c++" \
#  runtimeDeps="" \
RUN microdnf -y update \
    && microdnf -y install --nodocs yum-utils \
    && dnf config-manager --set-enabled ol8_codeready_builder \
    && dnf install -y --nodocs \
        libyaml-devel \
        wget \
        git-core \
        zlib \
        zlib-devel \
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
        sqlite-devel \
        net-tools \
    && wget -O /tmp/v20230717.tar.gz "https://github.com/rbenv/ruby-build/archive/refs/tags/v20230717.tar.gz" \
    && cd /tmp && tar -xzf v20230717.tar.gz \
    && PREFIX=/usr/local /tmp/ruby-build-20230717/install.sh \
    && ruby-build 3.0.6 /usr/local/ruby \
    && gem install oj -N -v 3.10.18 \
    && gem install cgi -N -v 0.3.1 --default \
    && gem install json -N -v 2.4.1 \
    && gem install async-http -N -v 0.54.0 \
    && gem install fluentd -N -v 1.14.5 \
    && gem install zlib -N -v 2.1.1 \
    && gem install bigdecimal -N -v 1.4.4 \
    && dpkgArch="amd64" \
    && gem install bundler -N --version 2.2.24 \
    && gem install rexml -N -v 3.2.5 \
    && gem install /tmp/fluent-plugin-oci-logging-1.0.2.gem -N \
    && bundle config silence_root_warning true \
    && bundle config set --local path '/fluentd/vendor/bundle' \
    && bundle install --gemfile=/fluentd/Gemfile \
    && gem sources --clear-all \
    && wget -O /usr/local/bin/tini "https://github.com/krallin/tini/releases/download/v$TINI_VERSION/tini-$dpkgArch" \
    && wget -O /usr/local/bin/tini.asc "https://github.com/krallin/tini/releases/download/v$TINI_VERSION/tini-$dpkgArch.asc" \
    && export GNUPGHOME="$(mktemp -d)" \
    && gpg --batch --keyserver hkp://keyserver.ubuntu.com --recv-keys 6380DC428747F6C393FEACA59A84159D7001A4E5 \
    && gpg --batch --verify /usr/local/bin/tini.asc /usr/local/bin/tini \
    && rm -r /usr/local/bin/tini.asc \
    && chmod +x /usr/local/bin/tini \
    && rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem \
    && rm -rf /fluentd/vendor/bundle/ruby/3.0.6/cache/* /fluentd/vendor/bundle/ruby/3.0.6/doc/* \
    && dnf -y autoremove \
        yum-utils \
        wget \
	git-core \
 #       zlib \
 #       zlib-devel \
        gcc-c++ \
        bzip2 \
        libtool \
        bison \
        libtool \
        make \
        libffi-devel \
        openssl-devel \
        autoconf \
        automake \
        readline-devel \
        sqlite-devel \
        patch \
        gcc \
        cpp \
        net-tools \
        libyaml-devel \
    && dnf clean all \
    && rm -rf /var/cache/yum /var/lib/rpm/* /var/cache/dnf /var/lib/dnf/* \
    && rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem \
    && rm -rf /usr/lib/locale/locale-archive \
    && rm -rf /usr/share/locale/* \
    && rm -rf /usr/share/doc/* /usr/include/* \
    && groupadd -r fluent && useradd -r -g fluent fluent \
    # for log storage (maybe shared with host)
    && mkdir -p /fluentd/log \
    # configuration/plugins path (default: copied from .)
    && mkdir -p /fluentd/etc /fluentd/plugins \
    && chown -R fluent /fluentd && chgrp -R fluent /fluentd \
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

# Environment variables
ENV FLUENTD_CONF="fluent.conf"
ENV FLUENTD_OPT=""

EXPOSE 24224 5140

# ENTRYPOINT to run fluentd as root for /var/log / /var/lib
ENTRYPOINT ["tini", "--", "/fluentd/entrypoint.sh"]
