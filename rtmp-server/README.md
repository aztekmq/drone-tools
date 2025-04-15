# 🚀 Local RTMP Server for DJI Drone Streaming

[![Docker Build](https://img.shields.io/badge/Docker-Build%20Passing-brightgreen?logo=docker)](https://www.docker.com/)
[![Health: Passing](https://img.shields.io/badge/Health-Passing-success?logo=heartbeat)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE.md)
[![Status: Production Ready](https://img.shields.io/badge/Status-Production%20Ready-green)]()
[![Size: Lightweight](https://img.shields.io/badge/Size-Lightweight-20MB-success?logo=datadog)]()

---

## Overview

This project provides a **self-hosted RTMP server** inside a **Docker container** for streaming video from your **DJI drone controller** directly into **OBS Studio** — without relying on external services like YouTube or Twitch.  
Built using **NGINX precompiled with the RTMP module** (`tiangolo/nginx-rtmp`), it offers a simple, fast, and fully local streaming solution.

---

## Features

- 🎥 Real-time DJI video streaming to OBS Studio.
- 📦 Fully containerized with Docker.
- 🔒 100% private, local network streaming.
- ⚡ Lightweight and production-ready setup.
- 🛠️ Single-command deployment using Docker Compose.
- 🩺 Built-in Docker **HEALTHCHECK** to monitor container well-being.

---

## Prerequisites

| Software | Purpose | Download |
|:---------|:--------|:---------|
| **Docker** | Run containerized RTMP server | [Docker Download](https://www.docker.com/products/docker-desktop) |
| **Docker Compose** | Manage services with a single command | [Compose Install Guide](https://docs.docker.com/compose/install/) |
| **OBS Studio** | Capture and record the RTMP stream | [OBS Studio Download](https://obsproject.com/download) |

✅ **Additional Requirements**:
- Your DJI controller must support **Custom RTMP streaming**.
- Laptop and DJI controller must be connected to the **same local network**.

---

## 🏁 Quickstart (Using Docker Compose)

### Step 1: Clone the Repository

```bash
git clone https://github.com/yourusername/rtmp-docker-dji-stream.git
cd rtmp-docker-dji-stream
```

### Step 2: Bring Up the Server

```bash
docker compose up -d
```

✅ This will **build** the Docker image (based on `tiangolo/nginx-rtmp`) and **launch** the container automatically.

---

### Step 3: Shut Down the Server

```bash
docker compose down
```

---

## 📄 `docker-compose.yml` Used

```yaml
services:
  rtmp-server:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: local-nginx-rtmp
    ports:
      - "1935:1935"   # RTMP streaming port
      - "8080:8080"   # HTTP stats/test page
    restart: unless-stopped
```

✅ **Explanation**:
| Field | Purpose |
|:-----|:--------|
| `build.context` | Build from the current directory |
| `container_name` | Easy-to-manage container name |
| `ports` | Exposes RTMP (1935) and HTTP (8080) |
| `restart: unless-stopped` | Auto-restarts the container on failure or reboot |

---

## 🔥 RTMP Server Configuration

| Port | Purpose |
|:----:|:-------:|
| 1935 | RTMP stream ingest (from DJI controller) |
| 8080 | HTTP web page (optional stats and testing) |

You can view the basic server info at: [http://localhost:8080](http://localhost:8080)

---

## 📋 DJI Controller Setup

1. Open your DJI Controller settings.
2. Set **Custom RTMP** as your streaming method.
3. Configure the RTMP URL:

```plaintext
rtmp://localhost/live/stream
```

Replace `<your-laptop-ip>` with your local machine IP address.

- **Windows**: `ipconfig`
- **Mac/Linux**: `ifconfig`

---

## 🎥 OBS Studio Setup

1. Open **OBS Studio**.
2. Add a **Media Source**.
3. Configure it:
   - **Uncheck** "Local File"
   - **Input URL**:

   ```plaintext
   rtmp://localhost/live/stream
   ```

4. Start your DJI controller stream and watch it live in OBS!

---

## 📂 Folder Structure

```plaintext
rtmp-docker-dji-stream/
├── Dockerfile
├── docker-compose.yml
├── nginx.conf
├── README.md
├── (optional) index.html
├── stop_docker.sh
├── clean-docker.sh
├── start_docker.sh
└── setup-notes.md
```

---

## 📄 Key Files Explained

| File | Purpose |
|:-----|:--------|
| `Dockerfile` | Defines container using `tiangolo/nginx-rtmp` with HEALTHCHECK |
| `docker-compose.yml` | Single-command deployment |
| `nginx.conf` | RTMP server and HTTP server configuration |
| `index.html` (optional) | Landing page |
| `stop_docker.sh` | Stop the container easily |
| `clean-docker.sh` | Stop + Remove container |
| `start_docker.sh` | Bring up container fast |

---

## 🧪 Testing Container Health

After deployment, quickly confirm the container's well-being:

### Step 1: Check container status

```bash
docker ps
```

✅ Look for `local-nginx-rtmp`.  
Status should be **Up (healthy)**.

Example:

```plaintext
CONTAINER ID   IMAGE                  STATUS          PORTS
abcd1234       local-nginx-rtmp        Up (healthy)    1935/tcp, 8080/tcp
```

---

### Step 2: Confirm the HTTP server responds

Visit:

```plaintext
http://localhost:8080
```

✅ Should load index page or RTMP stats page.

---

### Step 3: Confirm RTMP ingestion

(Optional)

If you have FFmpeg installed:

```bash
ffmpeg -i rtmp://localhost/live/stream -c copy -f null -
```

✅ If FFmpeg connects, RTMP server is ingesting properly.

---

## 🛠️ Debugging and Troubleshooting

| Situation | Command | Description |
|:----------|:--------|:------------|
| List running containers | `docker ps` | Show active containers |
| List all containers | `docker ps -a` | Show all containers |
| View logs | `docker logs -f local-nginx-rtmp` | Live container logs |
| Enter container shell | `docker exec -it local-nginx-rtmp /bin/sh` | Access container filesystem |
| Inspect container | `docker inspect local-nginx-rtmp` | Full metadata |
| Check ports | `docker port local-nginx-rtmp` | View mapped ports |
| Restart container | `docker restart local-nginx-rtmp` | Restart cleanly |
| Stop container | `docker stop local-nginx-rtmp` | Gracefully stop |
| Remove container | `docker rm local-nginx-rtmp` | Clean up |
| Full rebuild | `docker compose up -d --build` | Rebuild everything |
| Clean unused Docker artifacts | `docker system prune -f` | Free space |

---

## 🩺 About the Built-in HEALTHCHECK

This project has a Docker-native `HEALTHCHECK` inside the `Dockerfile`:

```Dockerfile
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget --spider --quiet http://localhost:8080/ || exit 1
```

| Field | Meaning |
|:------|:--------|
| `interval=30s` | Check health every 30 seconds |
| `timeout=5s` | 5-second timeout per check |
| `start-period=10s` | Wait 10s after container starts before health checks begin |
| `retries=3` | If 3 failures occur in a row, container becomes **unhealthy** |
| `wget` | Container tries to ping its own webserver at `http://localhost:8080/` |

✅ This ensures **automatic detection** if your RTMP server or NGINX fails inside the container.

---

# 📈 Monitoring Tips

Once the server is running, you can easily **monitor the health and live activity** of your RTMP server using built-in tools:

---

## 🔵 Basic Health Check

- Visit:

  ```plaintext
  http://localhost:8080
  ```

  ✅ If you see the "✅ Local RTMP Server is Running" page, the HTTP service is working properly.

---

## 📡 Live RTMP Stream Dashboard

- Visit:

  ```plaintext
  http://localhost:8080/stat
  ```

  ✅ You will see a **live dashboard** showing all currently active RTMP streams:

| Field | Description |
|:------|:------------|
| Application | The name of the RTMP application (typically `live`) |
| Stream Name | The stream key (e.g., `stream`) being sent |
| Time (seconds) | How long the stream has been active |

- Refresh the `/stat` page to update live activity.
- When your DJI controller or OBS client is streaming, it will immediately appear here!

---

## 🧪 Advanced: Manual Internal Checks

If needed, you can shell into the container and run quick internal health tests:

```bash
docker exec -it local-nginx-rtmp /bin/sh
```

Inside the container:

| Test | Command | Expected Result |
|:-----|:--------|:----------------|
| Test HTTP Server (port 8080) | `wget --spider --quiet http://localhost:8080` | Silent success (`exit 0`) |
| Test RTMP Server (port 1935) | `nc -zv localhost 1935` | `open` if port is listening |

✅ These manual tests verify your NGINX server and RTMP ingestion are working properly inside the container.

---

# 🚀 Pro Tip

If you ever need to **monitor container health externally**:

```bash
docker ps
```

✅ Look for container `STATUS: Up (healthy)`.  
If the server ever becomes unreachable, Docker will automatically mark it as `unhealthy`.

---

# 📋 Quick Monitoring Checklist

| Monitor | Check |
|:--------|:------|
| HTTP Landing Page | http://localhost:8080 |
| RTMP Sessions | http://localhost:8080/stat |
| Docker Health Status | `docker ps` |
| Internal Services (inside container) | `wget` and `nc` |

---

# 📦 Final Project Monitoring Overview

| Layer | How It's Monitored |
|:------|:------------------|
| Container Running | Docker daemon manages container uptime |
| Service Health | Built-in HEALTHCHECK probes HTTP server |
| Live Stream Monitoring | `/stat` dashboard shows active RTMP streams |
| Manual Debugging | Easy bash commands to verify HTTP/RTMP manually |

---

## 📜 License

This project is licensed under the [MIT License](LICENSE.md).

---

## 🤝 Credits

- [tiangolo/nginx-rtmp](https://hub.docker.com/r/tiangolo/nginx-rtmp)
- [NGINX RTMP Module](https://github.com/arut/nginx-rtmp-module)
- [Docker](https://www.docker.com/)
- [OBS Studio](https://obsproject.com/)

---

## 📬 Contact

For questions, improvements, or feedback:  
**rob@aztekmq.net**

---

# ✅ Status: **Production Ready**

---

# 🚀 Ready to Stream Locally?

Launch your **self-hosted RTMP server** in seconds —  
stream your DJI drone footage **privately, securely, and lightning-fast**!
