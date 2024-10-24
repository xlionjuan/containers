FROM fedora:latest

LABEL org.opencontainers.image.description="Simple container image just for create RPM repo."

RUN dnf -y install createrepo_c reprepro dpkg jq wget rpm-sign gnupg git \
    && dnf clean all \
    && rm -rf /var/cache/dnf /var/log/dnf.log /var/lib/rpm/__db*