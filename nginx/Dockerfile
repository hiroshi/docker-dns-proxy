FROM debian:jessie

RUN apt-get update && \
  apt-get install --no-install-recommends -y nginx && \
  apt-get autoremove -y && \
  rm -rf /var/lib/apt/lists/*
