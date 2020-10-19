FROM python:3.8.6-alpine

ARG ELASTALERT_VERSION=0.2.4

RUN apk --update upgrade && \
    apk add gcc libffi-dev musl-dev python3-dev openssl-dev tzdata libmagic && \
    rm -rf /var/cache/apk/*

RUN pip install elastalert==${ELASTALERT_VERSION} && \
    apk del gcc libffi-dev musl-dev openssl-dev

RUN mkdir -p /opt/elastalert && \
    mkdir -p /opt/elastalert/alert_rules && \
    echo "#!/bin/sh" >> /opt/elastalert/run.sh && \
    echo "set -e" >> /opt/elastalert/run.sh && \
    echo "elastalert-create-index --config /opt/elastalert/elastalert_config.yaml" >> /opt/elastalert/run.sh && \
    echo "exec elastalert --config /opt/elastalert/elastalert_config.yaml --verbose \"\$@\"" >> /opt/elastalert/run.sh && \
    chmod +x /opt/elastalert/run.sh

COPY elastalert_config.yaml /opt/elastalert/elastalert_config.yaml
COPY alert_rules/* /opt/elastalert/alert_rules/

WORKDIR /opt/elastalert
ENTRYPOINT ["/opt/elastalert/run.sh"]
