# Container image for running Buildah
#
# Based on the upstream Buildah image configuration:
# https://github.com/containers/image_build/tree/main/buildah

FROM docker.io/library/debian:trixie-slim

ARG DEBIAN_FRONTEND=noninteractive

# Install Buildah and the tools that it needs to run inside a container
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        buildah \
        ca-certificates \
        fuse-overlayfs \
        libcap2-bin \
        netavark \
        uidmap \
        amazon-ecr-credential-helper \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Create an unprivileged user with a range of subordinate IDs for rootless builds
RUN useradd --create-home --shell /bin/bash build \
    && echo "build:1:999" > /etc/subuid \
    && echo "build:1001:64535" >> /etc/subuid \
    && cp /etc/subuid /etc/subgid \
    && chmod 0755 /usr/bin/newuidmap /usr/bin/newgidmap \
    && setcap cap_setuid=ep /usr/bin/newuidmap \
    && setcap cap_setgid=ep /usr/bin/newgidmap \
    && mkdir -p /run/user/1000 \
    && chown build:build /run/user/1000 \
    && chmod 0700 /run/user/1000

# Use the host network and cgroups for RUN steps, because nested namespaces
# are often unavailable inside containers
RUN mkdir -p /etc/containers \
    && printf '%s\n' \
        '[containers]' \
        'netns = "host"' \
        'userns = "host"' \
        'ipcns = "host"' \
        'utsns = "host"' \
        'cgroupns = "host"' \
        'cgroups = "disabled"' \
        'log_driver = "k8s-file"' \
        '' \
        '[engine]' \
        'cgroup_manager = "cgroupfs"' \
        'events_logger = "file"' \
        > /etc/containers/containers.conf

# Use overlay storage with fuse-overlayfs, so that nested builds work
RUN printf '%s\n' \
        '[storage]' \
        'driver = "overlay"' \
        'runroot = "/run/containers/storage"' \
        'graphroot = "/var/lib/containers/storage"' \
        '' \
        '[storage.options.overlay]' \
        'mount_program = "/usr/bin/fuse-overlayfs"' \
        'mountopt = "nodev,fsync=0"' \
        > /etc/containers/storage.conf \
    && mkdir -p /home/build/.config/containers \
    && sed -e 's|^runroot.*|runroot = "/run/user/1000/containers"|' \
        -e 's|^graphroot.*|graphroot = "/home/build/.local/share/containers/storage"|' \
        /etc/containers/storage.conf > /home/build/.config/containers/storage.conf \
    && mkdir -p /home/build/.local/share/containers/storage /var/lib/containers/storage \
    && chown -R build:build /home/build

# Isolate RUN steps with chroot
ENV BUILDAH_ISOLATION=chroot

# Declare storage as volumes, to avoid overlay-on-overlay issues
VOLUME ["/var/lib/containers", "/home/build/.local/share/containers"]

CMD ["buildah", "--version"]
