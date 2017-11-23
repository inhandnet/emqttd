FROM ubuntu:trusty

ENV LC_CTYPE C.UTF-8
ENV LC_ALL C.UTF-8
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update \
    && apt-get install -y openssl \
    && rm -rf /var/lib/apt/lists/*

COPY emqttd /emqttd
WORKDIR /emqttd

ENTRYPOINT ["bin/emqttd"]
CMD ["console", "--noshell"]
