---
title: Systemd
---
# How to start a Warthog node as a systemd service 
**NOTE**: This is only supported in Linux distros that use `systemctl` (Debian, Ubuntu, Mint etc.).
* Enable auto restart when logged off (only need to do this once per machine): `$ sudo loginctl enable-linger $USER`
* Create the directory structure: `$ mkdir -p ~/.config/systemd/$USER`
* Create a file with the following content at that directory, file path: `~/.config/systemd/$USER/warthog.service`
    ```
    [Unit]
    Description=Warthog Node

    [Service]
    WorkingDirectory=<DIRECTORY WHERE YOU PLACED THE WARTHOG NODE EXECUTABLE>
    ExecStart=<PATH TO WARTHOG EXECUTABLE>
    Restart=always

    [Install]
    WantedBy=multi-user.target
    ```
* Reload systemctl daemon: `systemctl --user daemon-reload`
* Enable auto restart for Warthog service: `systemctl --user enable warthog`
* Start for Warthog service: `systemctl --user start warthog`
* Check status for Warthog service: `systemctl --user status warthog`

# How to start a public node
You need to have a static IP. Do as above but append the `--enable-public` flag at startup, i.e. use this file content:
```
[Unit]
Description=Warthog Node

[Service]
WorkingDirectory=<DIRECTORY WHERE YOU PLACED THE WARTHOG NODE EXECUTABLE>
ExecStart=<PATH TO WARTHOG EXECUTABLE> --enable-public
Restart=always

[Install]
WantedBy=multi-user.target
```

`--enable-public` is shorthand for `--publicrpc=0.0.0.0:3001`. It exposes a filtered subset of the API on port 3001 (critical admin endpoints are not exposed). The default RPC port (3000) is full-access and should never be exposed to the internet.
