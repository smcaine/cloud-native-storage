FROM python:3.8.2-slim
# ARG RALLY_VERSION
# ARG RALLY_LICENSE

ENV RALLY_RUNNING_IN_DOCKER True

RUN apt-get -y update && \
    apt-get install -y curl git gcc pbzip2 pigz && \
    apt-get -y upgrade && \
    rm -rf /var/lib/apt/lists/*

RUN groupadd --gid 1000 rally && \
    useradd -d /rally -m -k /dev/null -g 1000 -N -u 1000 -l -s /bin/bash rally

RUN pip3 install esrally

WORKDIR /rally

USER rally

ENV PATH=/rally/venv/bin:$PATH

LABEL org.label-schema.schema-version="1.0" \
  org.label-schema.vendor="Elastic" \
  org.label-schema.name="rally" \
  # org.label-schema.version="${RALLY_VERSION}" \
  org.label-schema.url="https://esrally.readthedocs.io/en/stable/" \
  org.label-schema.vcs-url="https://github.com/elastic/rally"

COPY --chown=rally:rally ./rally.ini /rally/.rally/rally.ini

ENTRYPOINT ["esrally"]