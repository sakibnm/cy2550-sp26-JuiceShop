+++
date = '2026-03-31T14:01:13-04:00'
draft = false
title = 'Installation'
+++
# Setup Guide — Installing OWASP Juice Shop with Docker

> **Purpose:** This guide walks you through installing and running the OWASP Juice Shop on your local computer using Docker. Complete this setup before attempting any of the challenges.

---

## What You Will Need

- A computer running **Windows 10/11**, **macOS**, or **Linux**
- **Docker Desktop** (Windows/macOS) or **Docker Engine** (Linux)
- At least **4 GB of free RAM** and **2 GB of free disk space**
- A modern web browser (Chrome, Firefox, or Edge recommended)
- An internet connection (for the initial download only — Juice Shop runs offline after that)

---

## Step 1 — Install Docker

If you already have Docker installed, skip to Step 2.

### Windows

1. Download **Docker Desktop** from [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
2. Run the installer and follow the prompts.
3. When asked, ensure **"Use WSL 2 instead of Hyper-V"** is checked (recommended).
4. Restart your computer if prompted.
5. After reboot, Docker Desktop should start automatically. Look for the whale icon in your system tray.
6. Open a terminal (PowerShell or Command Prompt) and verify the installation:

   ```bash
   docker --version
   ```

   You should see output like: `Docker version 27.x.x, build ...`

### macOS

1. Download **Docker Desktop** from [https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)
   - Choose the correct version for your chip: **Apple Silicon (M1/M2/M3/M4)** or **Intel**.
2. Open the `.dmg` file and drag Docker to your Applications folder.
3. Launch Docker from Applications. Grant any permissions it requests.
4. Wait for Docker Desktop to finish starting (the whale icon in the menu bar will stop animating).
5. Open **Terminal** and verify:

   ```bash
   docker --version
   ```

### Linux (Ubuntu/Debian)

1. Update your package index and install prerequisites:

   ```bash
   sudo apt-get update
   sudo apt-get install ca-certificates curl gnupg
   ```

2. Add Docker's official GPG key and repository:

   ```bash
   sudo install -m 0755 -d /etc/apt/keyrings
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
   sudo chmod a+r /etc/apt/keyrings/docker.gpg

   echo \
     "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
     $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
     sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

3. Install Docker Engine:

   ```bash
   sudo apt-get update
   sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
   ```

4. Add your user to the `docker` group (so you don't need `sudo` for every command):

   ```bash
   sudo usermod -aG docker $USER
   ```

   Log out and log back in for this to take effect.

5. Verify:

   ```bash
   docker --version
   ```

---

## Step 2 — Pull the Juice Shop Image

Open a terminal and download the latest Juice Shop Docker image:

```bash
docker pull bkimminich/juice-shop
```

This will download approximately **400–500 MB**. Wait for it to complete. You should see output ending with:

```
Status: Downloaded newer image for bkimminich/juice-shop:latest
```

---

## Step 3 — Run the Juice Shop

Start the container:

```bash
docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop
```

Here's what each flag does:

| Flag | Meaning |
|------|---------|
| `-d` | Run in the background (detached mode) |
| `-p 3000:3000` | Map port 3000 on your machine to port 3000 in the container |
| `--name juice-shop` | Give the container a friendly name |
| `bkimminich/juice-shop` | The Docker image to run |

---

## Step 4 — Verify It's Running

1. Check that the container is running:

   ```bash
   docker ps
   ```

   You should see a row with `bkimminich/juice-shop` and a status of `Up`.

2. Open your browser and navigate to:

   ```
   http://localhost:3000
   ```

3. You should see the **OWASP Juice Shop** storefront. If you see a product catalog with items like "Apple Juice" and "Orange Juice," the setup is complete.

   > **Note:** It may take 10–20 seconds for the application to fully start. If you see a connection error, wait a moment and refresh.

---

## Step 5 — Set Up Your Browser (Recommended)

For the best experience during the challenges, configure your browser:

### Open Developer Tools

- **Chrome/Edge:** Press `F12` or `Ctrl+Shift+I` (Windows/Linux) / `Cmd+Option+I` (macOS)
- **Firefox:** Press `F12` or `Ctrl+Shift+I` (Windows/Linux) / `Cmd+Option+I` (macOS)

Keep the **Network**, **Console**, and **Elements** tabs accessible — you'll use them frequently.

### Disable Browser Caching (Optional but Recommended)

In Developer Tools → Network tab, check the **"Disable cache"** checkbox. This ensures you always see fresh responses from the Juice Shop, which is important when testing injections.

### Install a Proxy Tool (Optional — Needed for Challenges 8 & 9)

For challenges that require intercepting and modifying HTTP requests, install one of the following:

- **Burp Suite Community Edition** (free): [https://portswigger.net/burp/communitydownload](https://portswigger.net/burp/communitydownload)
- **OWASP ZAP** (free & open source): [https://www.zaproxy.org/download/](https://www.zaproxy.org/download/)

Both tools act as a local proxy that sits between your browser and the Juice Shop, letting you intercept, inspect, and modify requests before they reach the server.

---

## Daily Usage — Starting and Stopping

### Stop the Juice Shop

```bash
docker stop juice-shop
```

### Start it again later

```bash
docker start juice-shop
```

### Restart (resets the database to a clean state)

```bash
docker restart juice-shop
```

> **Important:** The Juice Shop resets its database every time it restarts. Your challenge progress is saved in your browser's local storage, but any data you injected (XSS payloads, modified products, etc.) will be wiped clean. This is by design — it makes the application self-healing.

### View logs (useful for debugging)

```bash
docker logs juice-shop
```

### Remove the container entirely

```bash
docker stop juice-shop
docker rm juice-shop
```

To run it again after removal, repeat the `docker run` command from Step 3.

---

## Tutorial Mode (Recommended for Classrooms)

If you are using this in a classroom and want to enforce a guided, progressive experience, start the Juice Shop in **Tutorial Mode**. This hides all challenges except those with built-in tutorials and unlocks them gradually by difficulty tier.

```bash
docker run -d -p 3000:3000 --name juice-shop -e "NODE_ENV=tutorial" bkimminich/juice-shop
```

The only difference is the `-e "NODE_ENV=tutorial"` flag, which sets an environment variable inside the container.

---

## Running Multiple Instances (For Lab Environments)

If each student needs their own isolated instance, run multiple containers on different ports:

```bash
docker run -d -p 3001:3000 --name juice-shop-student1 bkimminich/juice-shop
docker run -d -p 3002:3000 --name juice-shop-student2 bkimminich/juice-shop
docker run -d -p 3003:3000 --name juice-shop-student3 bkimminich/juice-shop
```

Student 1 accesses `http://localhost:3001`, Student 2 accesses `http://localhost:3002`, and so on.

For larger classes, consider using **Docker Compose** with a configuration file:

```yaml
# docker-compose.yml
services:
  student1:
    image: bkimminich/juice-shop
    ports:
      - "3001:3000"
  student2:
    image: bkimminich/juice-shop
    ports:
      - "3002:3000"
  student3:
    image: bkimminich/juice-shop
    ports:
      - "3003:3000"
```

Run all instances at once:

```bash
docker compose up -d
```

Stop all instances:

```bash
docker compose down
```

---

## Troubleshooting

### "Port 3000 is already in use"

Another application is using port 3000. Either stop that application or use a different port:

```bash
docker run -d -p 8080:3000 --name juice-shop bkimminich/juice-shop
```

Then access the Juice Shop at `http://localhost:8080` instead.

### "Cannot connect to the Docker daemon"

Docker Desktop is not running. Start it from your Applications (macOS), Start Menu (Windows), or run `sudo systemctl start docker` (Linux).

### "docker: Error response from daemon: Conflict. The container name is already in use"

A container with the name `juice-shop` already exists. Remove it first:

```bash
docker rm juice-shop
```

Or choose a different name in the `--name` flag.

### The page loads but looks broken or shows errors

Try a hard refresh (`Ctrl+Shift+R` / `Cmd+Shift+R`) or open in an incognito/private window. Browser extensions (especially ad blockers or security plugins) can sometimes interfere.

### I want to start completely fresh

Remove the container and run a new one:

```bash
docker stop juice-shop
docker rm juice-shop
docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop
```

This gives you a clean database and a fresh application state.

---

## Quick Reference Card

| Action | Command |
|--------|---------|
| Pull the image | `docker pull bkimminich/juice-shop` |
| Start (first time) | `docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop` |
| Start (tutorial mode) | `docker run -d -p 3000:3000 --name juice-shop -e "NODE_ENV=tutorial" bkimminich/juice-shop` |
| Stop | `docker stop juice-shop` |
| Start again | `docker start juice-shop` |
| Restart (reset DB) | `docker restart juice-shop` |
| View logs | `docker logs juice-shop` |
| Remove container | `docker stop juice-shop && docker rm juice-shop` |
| Check running containers | `docker ps` |
| Access in browser | `http://localhost:3000` |

---

## You're Ready!

Once you can see the Juice Shop storefront at `http://localhost:3000`, proceed to the challenges:

**[→ Challenge Index](index.md)**

---

[← Back to Index](index.md)
