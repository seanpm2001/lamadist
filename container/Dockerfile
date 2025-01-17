# syntax=docker/dockerfile:1
# SPDX-License-Identifier: Apache-2.0

FROM ubuntu:22.04

SHELL ["/bin/bash", "-o", "errexit", "-o", "nounset", "-o", "pipefail", "-c"]

ENV DEBIAN_FRONTEND noninteractive

# Required Packages for the Host Development System
# http://www.yoctoproject.org/docs/latest/mega-manual/mega-manual.html#required-packages-for-the-host-development-system
COPY packages.txt /root/packages.txt
RUN --mount=type=cache,target=/var/lib/apt \
    --mount=type=cache,target=/var/cache/apt \

    # Update APT lists
    apt-get update &&\
    apt-get -y upgrade &&\

    # Configure default locale.
    apt-get install --no-install-recommends -y locales &&\
    printf 'C\nC.UTF-8 UTF-8\nen_US.UTF-8 UTF-8\n' > /etc/locale.gen && locale-gen &&\
    update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 LANGUAGE=en_US:en &&\
    export LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 LANGUAGE=en_US:en &&\

    # Install base Yocto Poky requirements
    grep -vE '^[[:blank:]]*(#|$)' /root/packages.txt | xargs apt-get install --no-install-recommends -y &&\

    # Use Bash as the default /bin/sh
    ln -sf /bin/bash /bin/sh &&\

    # Create a non-root user that will perform the actual build
    id builder 2> /dev/null || useradd --uid 1000 --shell /bin/bash --create-home builder &&\
    echo 'builder ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers.d/builder &&\
    chmod 0660 /etc/sudoers.d/builder &&\

    # Cleanup
    apt-get autoclean &&\
    rm -rf /tmp/* /var/tmp/*

# We need these for consistent sorting, formating, etc., but they cannnot
# be set until after the locales are generated.  Yocto also requires them.
ENV LANGUAGE en_US:en
ENV LANG     en_US.UTF-8
ENV LC_ALL   en_US.UTF-8

# Install kas and other Python packages
COPY requirements.txt /root/requirements.txt
RUN --mount=type=cache,target=/root/.cache \

    # Install out-of-tree Python packages
    pip install --no-cache-dir --prefer-binary --ignore-installed -r /root/requirements.txt &&\

    # Cleanup
    rm -rf /tmp/* /var/tmp/*

USER builder
WORKDIR /home/builder
ENTRYPOINT ["/bin/dumb-init", "/usr/local/bin/kas"]
CMD ["--help"]

# NOTE: Uncomment these for debugging
#USER root
#WORKDIR /root
#ENTRYPOINT ["/bin/dumb-init", "/bin/bash"]
#CMD ['-l']
