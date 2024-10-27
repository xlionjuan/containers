FROM fedora:latest

RUN dnf -y install createrepo_c reprepro dpkg jq wget rpm-sign gnupg git \
    && dnf clean all \
    && rm -rf /var/cache/dnf /var/log/dnf.log /var/lib/rpm/__db*