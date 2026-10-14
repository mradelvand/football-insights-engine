FROM mcr.microsoft.com/devcontainers/python:1-3.12-bookworm
# The base image ships a Yarn apt source with an expired key, which breaks apt-get update in features.
RUN rm -f /etc/apt/sources.list.d/yarn.list
