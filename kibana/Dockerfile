#
# ** THIS IS AN AUTO-GENERATED FILE **
#

################################################################################
# Build stage 0
# Extract Kibana and make various file manipulations.
################################################################################
FROM ubuntu:focal AS prep_files

# Install toolchain to build dumb-init
RUN apt update && apt install -y build-essential curl

RUN mkdir /usr/share/kibana
WORKDIR /usr/share/kibana

RUN curl -sL https://artifacts.elastic.co/downloads/kibana/kibana-7.8.0-linux-x86_64.tar.gz | tar --strip-components=1 -zxf -
# replace amd64 node with arm64
RUN rm -rf /usr/share/kibana/node/* && \
    curl -sL https://nodejs.org/dist/v10.21.0/node-v10.21.0-linux-arm64.tar.gz | tar -C /usr/share/kibana/node/ --strip-components=1 -xzf -

# compile dumb-init from sources
RUN mkdir -p /opt/dumb-init
RUN curl -sL https://github.com/Yelp/dumb-init/archive/v1.2.2.tar.gz | tar -C /opt/dumb-init --strip-components=1 -zxf -
RUN cd /opt/dumb-init && make && cd -

# Ensure that group permissions are the same as user permissions.
# This will help when relying on GID-0 to run Kibana, rather than UID-1000.
# OpenShift does this, for example.
# REF: https://docs.openshift.org/latest/creating_images/guidelines.html
RUN chmod -R g=u /usr/share/kibana
RUN find /usr/share/kibana -type d -exec chmod g+s {} \;

################################################################################
# Build stage 1
# Copy prepared files from the previous stage and complete the image.
################################################################################
FROM ubuntu:focal
EXPOSE 5601

# Add Reporting dependencies.
RUN apt update && apt install -y dialog fontconfig freetype2-demos libreadline8 curl
# Use bash as default shell
RUN rm /usr/bin/sh && ln -s /usr/bin/bash /usr/bin/sh

# Bring in Kibana from the initial stage.
COPY --from=prep_files --chown=1000:0 /usr/share/kibana /usr/share/kibana
# Bring in dumb-init from the initial stage.
COPY --from=prep_files --chown=1000:0 /opt/dumb-init/dumb-init  /usr/local/bin/dumb-init

WORKDIR /usr/share/kibana
RUN ln -s /usr/share/kibana /opt/kibana

ENV ELASTIC_CONTAINER true
ENV PATH=/usr/share/kibana/bin:$PATH

# Set some Kibana configuration defaults.
COPY --chown=1000:0 config/kibana.yml /usr/share/kibana/config/kibana.yml

# Add the launcher/wrapper script. It knows how to interpret environment
# variables and translate them to Kibana CLI options.
COPY --chown=1000:0 bin/kibana-docker /usr/local/bin/

RUN chmod a+x /usr/local/bin/kibana-docker

# Ensure gid 0 write permissions for OpenShift.
RUN chmod g+ws /usr/share/kibana && find /usr/share/kibana -gid 0 -and -not -perm /g+w -exec chmod g+w {} \;

# Provide a non-root user to run the process.
RUN groupadd --gid 1000 kibana && useradd -u 1000 -g 1000 -d /usr/share/kibana kibana
USER kibana

LABEL org.label-schema.schema-version="1.0" org.label-schema.vendor="Elastic" org.label-schema.name="kibana" org.label-schema.version="7.8.0" org.label-schema.url="https://www.elastic.co/products/kibana" org.label-schema.vcs-url="https://github.com/elastic/kibana" org.label-schema.license="Elastic License" org.label-schema.usage="https://www.elastic.co/guide/en/kibana/index.html" org.label-schema.build-date="2020-06-14T20:39:23.401Z" license="Elastic License"

ENTRYPOINT ["/usr/local/bin/dumb-init", "--"]

CMD ["/usr/local/bin/kibana-docker"]
