# Copyright (c) 2016-2018 ForgeRock AS. Use of this source code is subject to the
# Common Development and Distribution License (CDDL) that can be found in the LICENSE file
# Docker image for Apache + AM HTTP agent.
FROM httpd:2.4
WORKDIR /opt

ENV AM_MAX_SHARED_POOL_SIZE 1048576000
ENV AM_DEFAULT_LOG_LEVEL All

# For testing purposes, you can build this image with ARGs to match your deployment
ARG AM_SERVER=http://openam.example.forgeops.com:80/openam
ARG AGENT_SERVER=http://apache-agent.example.forgeops.com

# Before building image, make sure you have apache 2.4 web agent in this dockerfile folder
ARG APACHE_AGENT=/var/tmp/agent.zip
COPY agent.zip /var/tmp/agent.zip

# Install needed sw
RUN apt-get update && apt-get install --no-install-recommends  -y unzip curl vim && \
      unzip -q /var/tmp/agent.zip && rm /var/tmp/agent.zip && \
      echo "password" >/var/tmp/pw.txt  && chmod 0400 /var/tmp/pw.txt && \
      rm -rf /var/lib/apt/lists/*

# Do a force installation. We are going to overwrite agent.conf file later on
RUN /opt/web_agents/apache24_agent/bin/agentadmin --s /usr/local/apache2/conf/httpd.conf \
        "$AM_SERVER"  \
        "$AGENT_SERVER" \
        "/" \
        "apache" \
        /var/tmp/pw.txt \
        --changeOwner --acceptLicence --forceInstall

WORKDIR /opt/web_agents/apache24_agent
COPY *.sh /opt/web_agents/

# Copy testing pages to htdocs
COPY neu.html /usr/local/apache2/htdocs/
COPY policy.html /usr/local/apache2/htdocs/

RUN echo "CoreDumpDirectory /tmp/" >> /usr/local/apache2/conf/httpd.conf
ENTRYPOINT ["/opt/web_agents/docker-entrypoint.sh"]

# Set entrypoint to docker-entrypoint.sh do to runtime installation/configuration steps
CMD ["/opt/web_agents/docker-entrypoint.sh"]
