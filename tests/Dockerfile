FROM mcr.microsoft.com/dotnet/sdk:9.0

ENV HOME=/root
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get install -y --no-install-recommends python3 && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /skill
