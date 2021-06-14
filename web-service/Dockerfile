FROM ubuntu:20.04

ENV CONSUL_VERSION=1.9.6
ENV HASHICORP_RELEASES=https://releases.hashicorp.com
ENV consulArch=amd64

ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y unzip curl jq wget supervisor apache2 && \
    rm -rf /var/lib/apt/lists/* 

RUN useradd --system --home /etc/consul.d --shell /bin/false consul

RUN mkdir -p /etc/consul.d && \
    mkdir -p /opt/consul && \
    chown consul:consul /etc/consul.d /opt/consul

RUN wget ${HASHICORP_RELEASES}/consul/${CONSUL_VERSION}/consul_${CONSUL_VERSION}_linux_${consulArch}.zip && \
    unzip -d /bin consul_${CONSUL_VERSION}_linux_${consulArch}.zip && \
    consul version

#COPY ./client.json /etc/consul.d/
COPY ./*.json /etc/consul.d/
COPY ./*.conf /etc/supervisor/conf.d/

EXPOSE 8500 8600 8600/udp

#CMD ["consul", "agent", "-config-dir", "/etc/consul.d"]
CMD ["/usr/bin/supervisord", "--nodaemon", "-c", "/etc/supervisor/supervisord.conf"]

