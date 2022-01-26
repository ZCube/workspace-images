ARG BASE_IMAGE
ARG BASE_TAG
FROM ${BASE_IMAGE}/full:${BASE_TAG}

# add your tools here
USER gitpod

RUN /home/gitpod/.cargo/bin/cargo install mdbook

USER root
