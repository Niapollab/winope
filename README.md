# 🛡️ Winope: Transparent Proxy Gateway for Podman

Route any container's entire network traffic (TCP, UDP, and DNS) through an encrypted [sing-box](https://sing-box.sagernet.org/) tunnel **automatically** with zero configuration required inside client containers.

---

## 🚀 Quick Setup

### 1. Place Config
```bash
mkdir -p ~/.config/winope
cp sing-box-config.json ~/.config/winope/sing-box-config.json
```

> [!WARNING]
> Fill in your proxy credentials in `sing-box-config.json`.

### 2. Install & Start Quadlet
```bash
mkdir -p ~/.config/containers/systemd
cp winope.network winope.container ~/.config/containers/systemd/

systemctl --user daemon-reload
systemctl --user start winope
```

Check status:
```bash
systemctl --user status winope
```

---

## 🪄 Route Any Container Through Winope

### Approach 1: Connect to `winope` Network (Recommended)
Client containers get their own independent IP and network namespace, with traffic automatically flowing through the gateway.

#### 💻 From the CLI:
```bash
podman run -it --rm --network winope docker.io/library/alpine:latest wget -qO- https://ifconfig.me
```

#### 📦 From a Quadlet (`.container` file):
```ini
[Unit]
Description=My App
After=winope.service
Requires=winope.service

[Container]
ContainerName=my-app
Image=docker.io/library/alpine:latest
Network=winope.network
Exec=sleep infinity

[Install]
WantedBy=default.target
```

> [!TIP]
> **Graceful Stop Ordering & Auto-Start**:
> In addition to `BindsTo=winope.service` on client containers, list client services in `winope.container` under `[Unit]`:
> ```ini
> Wants=my-app.service ...
> Before=my-app.service ...
> ```
> - `Wants=`: Starts client containers automatically whenever `winope` starts.
> - `Before=`: Ensures systemd stops clients before stopping or recreating `winope`.

---

### Approach 2: Shared Network Namespace (`container:winope`)
Share the exact network stack of `winope`.

#### 💻 From the CLI:
```bash
podman run -it --rm --network container:winope docker.io/library/alpine:latest wget -qO- https://ifconfig.me
```

#### 📦 From a Quadlet (`.container` file):
Just set `Network=container:winope` and bind the lifecycle with `BindsTo=winope.service` (this ensures the dependent container cleanly shuts down before `winope`, avoiding `container ... has dependent containers which must be removed before it` errors):
```ini
[Unit]
Description=My App
After=winope.service
BindsTo=winope.service

[Container]
ContainerName=my-app
Image=docker.io/library/alpine:latest
Network=container:winope
Exec=sleep infinity

[Install]
WantedBy=default.target
```

> [!TIP]
> **Graceful Stop Ordering & Auto-Start**:
> In addition to `BindsTo=winope.service` on client containers, list client services in `winope.container` under `[Unit]`:
> ```ini
> Wants=my-app.service ...
> Before=my-app.service ...
> ```
> - `Wants=`: Starts client containers automatically whenever `winope` starts.
> - `Before=`: Ensures systemd stops clients before stopping or recreating `winope`.

---

## 🔍 Verification

### Check Public IP (Egress)
```bash
podman run --rm --network winope alpine wget -qO- https://ifconfig.me
```

### Check DNS Resolution
```bash
# Public resolution (encrypted DoH through tunnel)
podman run --rm --network winope alpine nslookup google.com

# Internal Podman container resolution
podman run --rm --network winope alpine nslookup winope
```
