ARG FROM=debian:bullseye-slim
FROM ${FROM}

ENV RUNNER_NAME=""
ENV RUNNER_WORK_DIRECTORY="_work"
ENV RUNNER_TOKEN=""
ENV RUNNER_REPOSITORY_URL=""
ENV RUNNER_LABELS=""
ENV RUNNER_ALLOW_RUNASROOT=true
ENV GITHUB_ACCESS_TOKEN=""
ENV AGENT_TOOLSDIRECTORY=/opt/hostedtoolcache

# Labels.
LABEL maintainer="me@tcardonne.fr" \
    org.label-schema.schema-version="1.0" \
    org.label-schema.build-date=$BUILD_DATE \
    org.label-schema.vcs-ref=$VCS_REF \
    org.label-schema.name="tcardonne/github-runner" \
    org.label-schema.description="Dockerized GitHub Actions runner." \
    org.label-schema.url="https://github.com/tcardonne/docker-github-runner" \
    org.label-schema.vcs-url="https://github.com/tcardonne/docker-github-runner" \
    org.label-schema.vendor="Thomas Cardonne" \
    org.label-schema.docker.cmd="docker run -it tcardonne/github-runner:latest"

RUN apt-get update && \
    apt-get install --assume-yes \
        curl \
        git \
        supervisor \
        jq && \
    rm --recursive --force /var/lib/apt/lists/* && \
    apt-get clean

RUN mkdir --parents /home/runner ${AGENT_TOOLSDIRECTORY}

WORKDIR /home/runner

ARG GH_RUNNER_VERSION
RUN set -x && \
    apt-get update && \
    packages="build-essential zlib1g-dev liblttng-ust0 libcurl4-openssl-dev openssh-client" && \
    apt-get install --assume-yes ${packages} && \
    GH_RUNNER_VERSION=${GH_RUNNER_VERSION:-$(curl --silent "https://api.github.com/repos/actions/runner/releases/latest" | jq -r .tag_name | sed -E 's/v([^"]+)/\1/')} && \
    curl --fail --show-error --location --remote-name https://github.com/actions/runner/releases/download/v${GH_RUNNER_VERSION}/actions-runner-linux-x64-${GH_RUNNER_VERSION}.tar.gz && \
    tar -zxf actions-runner-linux-x64-${GH_RUNNER_VERSION}.tar.gz && \
    rm actions-runner-linux-x64-${GH_RUNNER_VERSION}.tar.gz && \
    ./bin/installdependencies.sh && \
    chown --recursive root: /home/runner && \
    apt-get purge --assume-yes --auto-remove ${packages} && \
    rm --recursive --force /var/lib/apt/lists/* && \
    apt-get clean

COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
RUN chmod 644 /etc/supervisor/conf.d/supervisord.conf

COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
