# Free & Open-Source VLSI / IC Design Environment on Windows
### Analog + Digital | WSL2 + Docker + IIC-OSIC-TOOLS | SKY130 & GF180 PDK

A complete step-by-step guide to setting up a **professional open-source full-stack IC design environment** on Windows — covering both **analog/mixed-signal** and **digital RTL-to-GDS** flows — using WSL2 and Docker. No license fees. No Linux machine required.

---

## What You Get

### Analog / Mixed-Signal Tools
| Tool | Purpose |
|---|---|
| **xschem** | Schematic entry |
| **ngspice** | SPICE simulation |
| **xyce** | Advanced SPICE simulator |
| **Magic VLSI** | Layout editor + DRC |
| **netgen** | LVS (Layout vs Schematic) |
| **klayout** | GDS viewer + DRC |
| **qucs-s** | RF / analog simulation GUI |
| **gaw3-xschem** | Waveform viewer |

### Digital / RTL Tools
| Tool | Purpose |
|---|---|
| **yosys** | RTL synthesis |
| **iverilog** | Verilog simulator |
| **ghdl** | VHDL simulator |
| **verilator** | Fast Verilog simulator |
| **gtkwave** | Digital waveform viewer |
| **openroad** | Place and route |
| **irsim** | Switch-level simulator |
| **slang** | SystemVerilog compiler |

### PDKs Included
| PDK | Process |
|---|---|
| **SKY130A** | SkyWater 130nm open-source PDK |
| **GF180MCU** | GlobalFoundries 180nm open-source PDK |

> **Total: 54 tools** pre-installed and pre-configured. Run `ls /foss/tools/` inside the container to see all of them.



## Phase 1 — Enable WSL2

### Step 1: Install WSL2

Open **PowerShell as Administrator** and run:

```powershell
wsl --install
```

This installs Ubuntu and enables WSL2 automatically. **Restart your PC** when prompted.

### Step 2: Verify WSL2 is active

After restart, open PowerShell and run:

```powershell
wsl --list --verbose
```

You should see `VERSION 2` next to Ubuntu. If it shows `1`, upgrade it:

```powershell
wsl --set-default-version 2
wsl --set-version Ubuntu 2
```

### Step 3: Open Ubuntu

Go to **Start Menu → Ubuntu** and set your username and password on first launch.

---

## Phase 2 — Install Docker Desktop

### Step 4: Download and install Docker Desktop

Download from: https://www.docker.com/products/docker-desktop/

After installation and restart, open Docker Desktop and configure:

```
Settings → General → Use WSL 2 based engine  (ON)
Settings → Resources → WSL Integration → Enable integration with my default WSL distro  (ON)
Settings → Resources → WSL Integration → Ubuntu toggle  (ON)
→ Apply & Restart
```

Enable auto-start so you never forget to launch it:

```
Settings → General → Start Docker Desktop when you log in
```

### Step 5: Add your user to the docker group

Open Ubuntu terminal and run:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

### Step 6: Verify Docker is working

```bash
docker run hello-world
```

Expected output: `Hello from Docker!` 

> **Troubleshooting:** If you get `permission denied while trying to connect to the Docker daemon socket` — make sure Docker Desktop is running (whale icon in Windows taskbar), then re-run `newgrp docker`.

---

## Phase 3 — Install VcXsrv (X11 Display Server for GUI)

Windows does not have a native X11 server. VcXsrv lets GUI windows from WSL/Docker appear on your Windows desktop.

### Step 7: Download and install VcXsrv

Download from: https://sourceforge.net/projects/vcxsrv/

### Step 8: Add Windows Firewall rule

Open **PowerShell as Administrator** and run:

```powershell
New-NetFirewallRule -DisplayName "VcXsrv" -Direction Inbound -Program "C:\Program Files\VcXsrv\vcxsrv.exe" -Action Allow
```

You should see `PrimaryStatus: OK` in the output. 

### Step 9: Find your WSL Host IP

> ⚠️ **Critical:** Do NOT use the IP from `/etc/resolv.conf` (e.g. `10.255.255.254`) — that is a WSL-internal fallback and will NOT work for X11.

**From WSL terminal:**
```bash
ip route show | grep default
```
Note the IP after `via`, for example: `172.21.192.1`

**OR from Windows CMD / PowerShell:**
```powershell
ipconfig
```
Look for this section:
```
Ethernet adapter vEthernet (WSL (Hyper-V firewall)):
   IPv4 Address: 172.21.192.1
```

### Step 10: Set the DISPLAY variable permanently

```bash
echo 'export DISPLAY=172.21.192.1:0.0' >> ~/.bashrc
source ~/.bashrc
```

> Replace `172.21.192.1` with your actual IP from Step 9.

### Step 11: Launch VcXsrv (XLaunch) — do this every session

```
Start Menu → XLaunch
→ Multiple windows           → Next
→ Start no client            → Next
→ Disable access control  → Finish
```

### Step 12: Test X11 is working

```bash
sudo apt install x11-apps -y
xclock
```

A clock window should appear on your Windows desktop. 
If not, re-check your IP (Step 9) and make sure XLaunch is running with "Disable access control" checked.

---

## Phase 4 — Pull the IIC-OSIC-TOOLS Docker Image

### Step 13: Create your design folder

```bash
mkdir -p ~/eda/design
```

### Step 14: Pull the image (one-time download, ~15-20 GB)

```bash
docker pull hpretl/iic-osic-tools:latest
```

This takes time — run it on a good connection (campus LAN recommended).

---

## Phase 5 — Create a Reusable Launch Script

### Step 15: Create the launch script

```bash
cat > ~/start_eda.sh << 'EOF'
#!/bin/bash
echo "Starting EDA Environment..."
docker run -it --rm \
  -e DISPLAY=172.21.192.1:0.0 \
  -v ~/eda/design:/foss/designs \
  --name iic-osic \
  hpretl/iic-osic-tools:latest
EOF
chmod +x ~/start_eda.sh
```

> Replace `172.21.192.1` with your actual Windows host IP.

### Step 16: Launch the environment

```bash
./start_eda.sh
```

Wait until you see:
```
[INFO] X server is ready.
[INFO] Waiting until one of the sub-processes stops...
```

A terminal window opens on your Windows desktop — you are now **inside the container**. 

---

## How to Open Each Tool

Once inside the container (prompt shows `/foss/designs >`), launch any tool by typing its name:

### Analog / Mixed-Signal Tools

```bash
xschem          # Schematic editor — opens GUI window
magic           # Layout editor + DRC — opens GUI window
klayout         # GDS viewer — opens GUI window
ngspice         # SPICE simulator (interactive CLI)
xyce            # Advanced SPICE simulator (CLI)
netgen          # LVS checker (CLI)
qucs-s          # RF/analog simulation GUI
```

### Digital / RTL Tools

```bash
gtkwave         # Waveform viewer — opens GUI window
yosys           # RTL synthesis (CLI)
iverilog        # Verilog compiler (CLI)
ghdl            # VHDL simulator (CLI)
verilator       # Fast Verilog simulator (CLI)
```

### Explore Everything Installed

```bash
# See all 54 tools
ls /foss/tools/

# Count them
ls /foss/tools/ | wc -l

# Check available PDKs
ls /foss/pdks/

# Browse ready-made demo circuits
ls /foss/examples/

# Verify a specific tool exists
which xschem
xschem --version
```

---

## Saving Your Work

> ⚠️ The `--rm` flag means the container is deleted when you close it. Only `/foss/designs/` is persistent.

- **Always save your files to `/foss/designs/`** inside the container
- This folder maps directly to `~/eda/design` on your Windows machine
- Files here **survive container restarts** 

---

## Relaunch Sequence (Every Session)

Every time you restart your laptop, follow this order:

| # | Action |
|---|---|
| 1 | Open **Docker Desktop** from Start Menu — wait for whale icon to appear in taskbar |
| 2 | Open **XLaunch** → Multiple windows → Start no client → ✅ Disable access control → Finish |
| 3 | Open **Ubuntu** terminal |
| 4 | Run `./start_eda.sh` |
| 5 | Inside container: type tool name to launch (e.g. `xschem`, `magic`, `klayout`) |

---

## Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| `docker: command not found` | Docker Desktop not running | Open Docker Desktop from Start Menu |
| `permission denied` (docker socket) | User not in docker group | Run `newgrp docker` |
| `Waiting for X server...` (stuck) | Wrong DISPLAY IP or VcXsrv not running | Start XLaunch with "Disable access control", check DISPLAY IP |
| `Can't open display` | Wrong Windows host IP | Run `ip route show \| grep default` in WSL, use that IP |
| `permission denied` on simulation folder | `/foss/examples/` is read-only | Copy files to `/foss/designs/` first |
| GUI window not appearing | VcXsrv firewall blocked | Run PowerShell as Admin, redo Step 8 |
| Container exits immediately | Missing `-it` flag | Use the launch script from Step 15 |

---

