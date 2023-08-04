ARG FLUENTD_IMG
FROM $FLUENTD_IMG

LABEL maintainer="Verrazzano developers <verrazzano_ww@oracle.com>"
USER root
WORKDIR /home/fluent
ENV PATH /fluentd/vendor/bundle/ruby/3.0.6/bin:$PATH
ENV GEM_PATH /fluentd/vendor/bundle/ruby/3.0.6
ENV GEM_HOME /fluentd/vendor/bundle/ruby/3.0.6
# skip runtime bundler installation
ENV FLUENTD_DISABLE_BUNDLER_INJECTION 1

COPY docker-image/v1.14/oraclelinux-elasticsearch7/Gemfile /fluentd/
COPY docker-image/v1.14/oraclelinux-elasticsearch7/Gemfile.lock /fluentd/
COPY docker-image/v1.14/oraclelinux-elasticsearch7/fluent-plugin-oci-logging-1.0.2.gem /tmp/
RUN buildDeps="sudo make gcc" \
  runtimeDeps="" \
    && microdnf -y update \
    && microdnf install -y \
    $buildDeps $runtimeDeps net-tools \
    && mircodnf clean all \
    && gem install bundler --version 2.2.24 \
    && gem install rexml -v 3.2.5 \
    && gem install /tmp/fluent-plugin-oci-logging-1.0.2.gem \
    && bundle config silence_root_warning true \
    && bundle config set --local path '/fluentd/vendor/bundle' \
    && bundle install --gemfile=/fluentd/Gemfile \
    && SUDO_FORCE_REMOVE=yes \
    && gem sources --clear-all \
    && rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem

# Copy configuration files
COPY docker-image/v1.14/oraclelinux-elasticsearch7/conf/fluent.conf /fluentd/etc/
COPY docker-image/v1.14/oraclelinux-elasticsearch7/conf/systemd.conf /fluentd/etc/
COPY docker-image/v1.14/oraclelinux-elasticsearch7/conf/kubernetes.conf /fluentd/etc/
COPY docker-image/v1.14/oraclelinux-elasticsearch7/conf/prometheus.conf /fluentd/etc/
RUN touch /fluentd/etc/disable.conf

# Copy plugins
COPY docker-image/v1.14/oraclelinux-elasticsearch7/plugins /fluentd/plugins/
COPY docker-image/v1.14/oraclelinux-elasticsearch7/entrypoint.sh /fluentd/entrypoint.sh

# COPY LICENSE and README files to the image
COPY LICENSE README.md /license/

# Environment variables
ENV FLUENTD_OPT=""
ENV FLUENTD_CONF="fluent.conf"

# Overwrite ENTRYPOINT to run fluentd as root for /var/log / /var/lib
ENTRYPOINT ["tini", "--", "/fluentd/entrypoint.sh"]
