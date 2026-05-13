# LibreChat-on-Fedora-44-Rootless-Podman-Quadlets-
LibreChat on Fedora: The Easy Way Guide

This guide describes how to run LibreChat on Fedora 44 using Podman and Quadlets. This setup fixes the Error 139 database crash.

# Project Structure

To keep things clean, separate the service files from your data.

This is where the service files from this GitHub go:
Quadlet Services: .config/containers/systemd/

Put your librechat.env and librechat.yaml here:
App Config: containers/librechat/config/

This is where your database is save:
Database Data: containers/librechat/mongodb/

This is where your chat images are save:
User Images: containers/librechat/images/

Step by Step Instructions

# 1. Create the folders
Open your terminal and run this command:

    mkdir -p ~/containers/librechat/config ~/containers/librechat/mongodb ~/containers/librechat/images ~/.config/containers/systemd

# 2. Prepare your config files
Put your librechat.yaml and librechat.env files into the containers/librechat/config/ folder.
Important: Your .env file must have this line:

    MONGO_URI=mongodb://librechat-db:27017/LibreChat

# 3. Set up the service files
Copy the .container and .network files from this GitHub into the .config/containers/systemd/ folder.
The database file includes two special lines that fix the Fedora 44 crash:

    Environment=GLIBC_TUNABLES=glibc.cpu.hwcaps=-shstk
    UserNS=keep-id

# 4. Start the chat
Run these two commands:

    systemctl --user daemon-reload
    systemctl --user start librechat-app.service

# 5. Now open your browser and go to 

    http://localhost:3080

# Notes
SELinux: Always append the :Z flag to your volume mounts (e.g., Volume=%h/containers/librechat/mongodb:/data/db:Z) to allow Fedora to label the files correctly for rootless access.

Kernel Tuning: For optimal performance, set vm.swappiness=1 and ensure Transparent Huge Pages (THP) are set to always on the host machine.

A special thanks to [@disi](https://github.com/disi) for the initial build and foundation of this project.
    https://github.com/disi/LibreChat_Podman/tree/main
