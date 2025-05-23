FROM ubuntu:22.04

ENV DEBIAN_FRONTEND=noninteractive \
    DEBCONF_NONINTERACTIVE_SEEN=true \
    LC_ALL=C.UTF-8 \
    LANG=C.UTF-8

# For Konnect API
RUN apt-get update
RUN apt-get install -y \
      curl \
      jq

# For deck CLI
RUN curl -sL https://github.com/kong/deck/releases/download/v1.40.2/deck_1.40.2_linux_amd64.tar.gz -o deck.tar.gz
RUN tar -xf deck.tar.gz -C /tmp
RUN install /tmp/deck /usr/local/bin/

# For inso CLI
RUN curl -sL https://github.com/Kong/insomnia/releases/download/core%4010.1.0/inso-linux-10.1.0.tar.xz -o inso.tar.xz
RUN apt-get install xz-utils
RUN tar -xf inso.tar.xz -C /tmp
RUN install /tmp/inso /usr/local/bin/

WORKDIR /root
CMD bash