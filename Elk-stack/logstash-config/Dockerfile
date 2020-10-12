FROM centos:7

# Install Java and the "which" command, which is needed by Logstash's shell
# scripts.
RUN yum install -y epel-release
RUN set -x && yum update -y && yum install -y java-11-openjdk-devel which python python-pip && yum clean all
RUN pip install envtpl

# Provide a non-root user to run the process.
RUN groupadd --gid 1000 logstash && \
    adduser --uid 1000 --gid 1000 --home-dir /usr/share/logstash --no-create-home logstash

# Add Logstash itself.
# https://github.com/hadolint/hadolint/wiki/DL4006
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN set -x && curl -Lo - https://artifacts.elastic.co/downloads/logstash/logstash-7.8.1.tar.gz | \
    tar zxf - -C /usr/share && \
    mv /usr/share/logstash-7.8.1 /usr/share/logstash && \
    chown --recursive logstash:logstash /usr/share/logstash/ && \
    chown -R logstash:root /usr/share/logstash && \
    chmod -R g=u /usr/share/logstash && \
    find /usr/share/logstash -type d -exec chmod g+s {} \; && \
    ln -s /usr/share/logstash /opt/logstash

WORKDIR /usr/share/logstash

ENV PATH=/usr/share/logstash/bin:$PATH

# Provide a minimal configuration, so that simple invocations will provide
# a good experience.
RUN mkdir -p config/aerospike_patterns/multi_pattern && \
    mkdir -p config/aerospike_patterns/patterns
COPY logstash.template.conf /usr/share/logstash/logstash.template.conf
COPY filebeat-template.json /usr/share/logstash/filebeat-template.json 
COPY aerospike-template.json /usr/share/logstash/aerospike-template.json
COPY rabbitmq-template.json /usr/share/logstash/rabbitmq-template.json
COPY orderapi-template.json /usr/share/logstash/orderapi-template.json
COPY mediaplanner-template.json /usr/share/logstash/mediaplanner-template.json
COPY celery-template.json /usr/share/logstash/celery-template.json
COPY sswa-template.json /usr/share/logstash/sswa-template.json

COPY event.template.json /usr/share/logstash/event.template.json
COPY error.template.json /usr/share/logstash/error.template.json
COPY access.template.json /usr/share/logstash/access.template.json

COPY pipelines.yml config/pipelines.yml
COPY logstash-full.yml config/logstash.yml
COPY log4j2.properties config/
COPY default.conf pipeline/logstash.conf
COPY aerospike_multiline_pattern config/aerospike_patterns/multi_pattern
COPY aerospike_patterns config/aerospike_patterns/patterns
RUN chown --recursive logstash:root config/ pipeline/

# Ensure Logstash gets a UTF-8 locale by default.
ENV LANG='en_US.UTF-8' LC_ALL='en_US.UTF-8'

# Place the startup wrapper script.
COPY docker-entrypoint /usr/local/bin/
RUN chmod 0755 /usr/local/bin/docker-entrypoint

USER 1000

COPY env2yaml /usr/local/bin/

EXPOSE 9600 5044 8088

LABEL org.label-schema.schema-version="1.0" \
  org.label-schema.vendor="Elastic" \
  org.label-schema.name="logstash" \
  org.label-schema.version="7.8.1" \
  org.label-schema.url="https://www.elastic.co/products/logstash" \
  org.label-schema.vcs-url="https://github.com/elastic/logstash" \
license="Elastic License"

ENTRYPOINT ["/usr/local/bin/docker-entrypoint"]
