# Project Zomboid Dedicated Server (Docker + Vagrant + FRP)

This project provides a dedicated Project Zomboid server infrastructure running inside a Vagrant-managed Virtual Machine (Ubuntu Focal64), containerized using Docker Compose, and exposed to the public internet via an FRP (Fast Reverse Proxy) tunnel to an Azure server.

---

## Project Architecture

```mermaid
graph TD
    Client[Zomboid Client] -->|UDP 16261/16262| AzureFRP[Azure FRPS Server 70.153.137.246]
    AzureFRP -->|FRP Tunnel| FRPC[FRP Client Container]
    FRPC -->|Local Network| PZ[Project Zomboid Container]
    
    subgraph Vagrant VM (Ubuntu Focal64)
        FRPC
        PZ
    end
    
    subgraph Host OS (Windows)
        Vagrant[Vagrant / VirtualBox]
    end
```

The infrastructure consists of:
1. **Windows Host**: Runs Vagrant and VirtualBox.
2. **Vagrant VM**: Ubuntu Focal 20.04 LTS allocated with 4 CPUs, 8 GB RAM, and 40 GB of disk space.
3. **Docker Compose**:
   - **`project-zomboid`**: Container based on the `afey/zomboid` image (using LinuxGSM) to run the game server.
   - **`frp-client`**: Container based on `snowdreamtech/frpc` which routes the local server ports to the public FRP server, making the game accessible from the internet without requiring router port-forwarding.

---

## Prerequisites

Before running the server, ensure you have the following software installed on the Windows Host:
- [Vagrant](https://www.vagrantup.com/downloads)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)
- [Git](https://git-scm.com/downloads)

---

## Getting Started

### 1. Start the VM
Open PowerShell in the project directory on your Windows Host and run:
```powershell
vagrant up
```
Note: The initial setup will download the Ubuntu box, expand the disk size, and automatically install Docker.

### 2. Start the Docker Containers
Access the VM via SSH and spin up the containers:
```powershell
vagrant ssh
```
Once connected to the VM terminal, navigate to the server directory and run:
```bash
cd zomboid-server
docker compose up -d
```

### 3. First-Time Admin Password Injection (Critical)
The `afey/zomboid` Docker image runs LinuxGSM inside a `tmux` session. On the first boot, the server pauses and prompts for an administrator password interactively.

Since this process runs detached in the background, you must inject the password manually. Run the helper script from the Windows Host:
```powershell
vagrant ssh -c "bash /vagrant/tmux_input.sh"
```
This script sends the default admin password (`madridnomor1`) directly to the `tmux` session inside the container.

---

## Default Configuration Details

- **Server Name**: `fastos-private`
- **Admin Username**: `admin`
- **Admin Password**: `madridnomor1`
- **Server Password (Client Join Password)**: `barcagakpernahucl`
- **FRP Server IP**: `70.153.137.246`
- **Game Ports**: `16261` (UDP) & `16262` (UDP)
- **XP Multiplier**: `3.0` (3x multiplier)

---

## Sandbox Configuration and Modding

All game configuration files are located inside the container at `/home/linuxgsm/Zomboid/Server/`.

### Modifying Sandbox Settings (Example: XP Multiplier)
1. Stop the container:
   ```bash
   docker compose down
   ```
2. Copy the sandbox configuration file (`fastos-private_SandboxVars.lua`) from the container to the VM or host.
3. Edit the value. For example, to adjust the XP multiplier:
   ```lua
   XpMultiplier = 3.0,
   ```
4. Copy the file back into the container and start the server again.

A detailed guide on modding and configuring sandbox settings is available in the guide document: [zomboid_server_guide.md](file:///C:/Users/Gerardo%20Ardianta/.gemini/antigravity/brain/04dcb552-79fe-4818-bb4c-05bb2bd849a6/zomboid_server_guide.md).

---

## Common Management Commands

Run these commands inside the `zomboid-server` directory inside the VM:

* **Check Container Status**:
  ```bash
  docker compose ps
  ```
* **View Server Logs**:
  ```bash
  docker logs -f project-zomboid
  ```
* **Stop the Server**:
  ```bash
  docker compose down
  ```
* **Access the Server Console (Interactive Tmux)**:
  If you need to enter commands directly into the Zomboid server console:
  ```bash
  docker exec -it -u linuxgsm project-zomboid tmux -L fastos-private-4dc30d7d attach -t fastos-private
  ```
  (Press `Ctrl+B` then `D` to detach from tmux without stopping the server).
