# 🛡️ Winope: Invisible Proxy Gateway for Podman

Route any container's entire traffic (TCP, UDP, and DNS) through an encrypted [Sing-box](https://sing-box.sagernet.org/) tunnel **automatically** with zero configuration inside client containers.

---

## 🚀 Quick Setup

### 1. Place Config
```bash
mkdir -p ~/.config/winope
cp sing-box-config.json ~/.config/winope/sing-box-config.json
```

> [!WARNING]
>
> (Fill in your proxy credentials in `sing-box-config.json`)

### 2. Install & Start Quadlet
```bash
mkdir -p ~/.config/containers/systemd
cp winope.network winope.container ~/.config/containers/systemd/

systemctl --user daemon-reload
systemctl --user start winope
```

---

## 🪄 Route Any Container Through Winope

### 💻 From the Command Line
```bash
podman run -it --rm --network container:winope docker.io/library/alpine:latest wget -qO- https://ifconfig.me
```

### 📦 From a Quadlet (`.container`)
Just add `Network=container:winope`:

```ini
[Unit]
Description=My App
After=winope.service
Wants=winope.service

[Container]
ContainerName=my-app
Image=docker.io/library/alpine:latest
Network=container:winope
Exec=sleep infinity

[Install]
WantedBy=default.target
```
