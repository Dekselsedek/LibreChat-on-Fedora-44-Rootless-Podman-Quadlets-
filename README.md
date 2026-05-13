# LibreChat-on-Fedora-44-Rootless-Podman-Quadlets-
This guide describes the setup of LibreChat on Fedora 44 using systemd Quadlets. This configuration ensures your containers run as native systemd services under a specific user context.

roubleshooting: Error 139 (SIGSEGV) on Fedora 44

During the deployment on Fedora 44, we identified a recurring crash with Exit Code 139 (Segmentation Fault) in the MongoDB container. While standard database optimizations are helpful, the root cause was tied to a conflict between the host's hardware security features and the container's glibc library.
The Root Cause

The primary culprit was SHSTK (Shadow Stack), a hardware-based security feature. In Fedora 44, the interaction between the host’s glibc and the MongoDB binary causes a memory conflict, resulting in a immediate SIGSEGV crash upon database activity.
The Definitive Fix

To resolve this, you must disable the hardware shadow stack capability for the database process and ensure correct user namespace mapping:

    Disable SHSTK: Add Environment=GLIBC_TUNABLES=glibc.cpu.hwcaps=-shstk to the [Service] section of your unit file.

    Maintain Permissions: Use UserNS=keep-id to ensure the container's User ID matches your host User ID, preventing Permission Denied errors on your data volumes.

Recommended Configuration for librechat-db.container

Incorporate these lines into your Quadlet or the resulting systemd service file:
Ini, TOML

[Service]
# Fixes Error 139 (SIGSEGV) by disabling hardware shadow stack conflicts
Environment=GLIBC_TUNABLES=glibc.cpu.hwcaps=-shstk
# Ensures host UID matches container UID for volume permissions
UserNS=keep-id
Restart=always

Additional Stability Checklist

    SELinux: Always append the :Z flag to your volume mounts (e.g., Volume=%h/containers/librechat/mongodb:/data/db:Z) to allow Fedora to label the files correctly for rootless access.

    Kernel Tuning: For optimal performance, set vm.swappiness=1 and ensure Transparent Huge Pages (THP) are set to always on the host machine.
