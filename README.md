# Tailscale Time Machine

Secure Apple Time Machine backups over Tailscale using Docker.

This project provides a Time Machine-compatible SMB backup target that is accessible anywhere through your Tailscale network. No port forwarding, public SMB exposure, or traditional VPN configuration is required.

## Features

* Secure remote Time Machine backups via Tailscale
* Docker Compose deployment
* Time Machine-compatible SMB shares
* No public Internet exposure
* Multiple isolated backup targets
* MagicDNS support
* Works across locations and networks
* Supports local disks, NAS storage, ZFS datasets, and mounted volumes

---

## How It Works

Traditional Time Machine network backups are usually limited to a local network:

```text
MacBook
   |
 Local LAN
   |
 SMB Server
```

This project allows Time Machine backups over a Tailscale tailnet:

```text
MacBook
   |
Tailscale
   |
Backup Server
   |
Docker
   |
Time Machine SMB Share
```

The SMB service remains private and is only accessible by authorized devices on your tailnet.

---

## Architecture

```text
┌─────────────┐
│   macOS     │
│ TimeMachine │
└──────┬──────┘
       │
       │ Tailscale
       │
┌──────▼──────┐
│  Tailnet    │
└──────┬──────┘
       │
┌──────▼────────────────────┐
│ Docker Host               │
│                            │
│  Tailscale Container       │
│           │                │
│           ▼                │
│      Caddy Layer4          │
│           │                │
│     SMB Containers         │
│           │                │
│      Backup Storage        │
└────────────────────────────┘
```

---

## Requirements

### Server

* Linux host
* Docker Engine
* Docker Compose
* Tailscale account
* Storage volume for backups

### Clients

* macOS
* Tailscale installed and connected

---

## Installation

### Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/tailscale-timemachine.git
cd tailscale-timemachine
```

### Configure Storage

Create storage locations for your Time Machine shares:

```bash
mkdir -p /tm-storage/tm-1
mkdir -p /tm-storage/tm-2
```

Adjust paths to match your environment.

### Configure Tailscale

Generate an auth key from:

https://login.tailscale.com/admin/settings/keys

Add the key to your environment variables or compose configuration.

### Review Configuration Files

This repository includes:

| File               | Purpose               |
| ------------------ | --------------------- |
| docker-compose.yml | Container definitions |
| config.json        | Share configuration   |
| smb.conf           | Samba configuration   |
| Dockerfile         | Build instructions    |

Review and customize these files before deployment.

### Start the Stack

```bash
docker compose up -d
```

Verify containers are running:

```bash
docker ps
```

---

## Connecting macOS

### Verify Tailscale Connectivity

```bash
tailscale status
```

Ensure the backup server appears online.

### Connect to the Share

In Finder:

```text
Go → Connect to Server
```

Enter:

```text
smb://backup-server
```

or

```text
smb://backup-server.tailnet.ts.net
```

depending on your MagicDNS configuration.

### Configure Time Machine

1. Open System Settings
2. General
3. Time Machine
4. Add Backup Disk
5. Select the SMB share
6. Authenticate if prompted
7. Begin backup

---

## Multiple Backup Targets

The stack supports multiple isolated Time Machine shares.

Example:

```text
tm-1
tm-2
tm-3
```

Each share can have:

* Dedicated storage
* Separate quotas
* Different users
* Independent access policies

This makes it suitable for:

* Families
* Small businesses
* Lab environments
* Multi-user deployments

---

## Storage Recommendations

Recommended:

* SSDs
* RAID arrays
* ZFS datasets
* Synology volumes
* TrueNAS datasets

Not recommended:

* SD cards
* USB flash drives
* Unreliable network mounts

Time Machine backups can become very large over time.

---

## Security

All connectivity is handled through Tailscale.

Benefits include:

* WireGuard encryption
* Device authentication
* Tailnet ACLs
* MagicDNS support
* No public SMB exposure

For additional security:

* Restrict access with Tailscale ACLs
* Use tagged devices
* Limit access to backup administrators

---

## Backup Recovery

Backups created with this project remain fully compatible with standard Apple recovery workflows.

Restore options include:

### Migration Assistant

Restore an entire Mac from a backup.

### Time Machine

Browse and restore individual files and folders.

---

## Troubleshooting

### Cannot Reach the Backup Server

Verify Tailscale connectivity:

```bash
tailscale status
```

### SMB Share Not Visible

Inspect container logs:

```bash
docker logs <container-name>
```

### Time Machine Refuses the Share

Verify:

* SMB share is accessible
* Write permissions are correct
* Storage volume has free space
* Time Machine support is enabled

### Check Container Status

```bash
docker compose ps
```

---

## Example Deployments

### Home Lab

```text
MacBook
   │
Tailscale
   │
Proxmox
   │
Docker
   │
Time Machine Storage
```

### Remote Backup Server

```text
MacBook
   │
Hotel Wi-Fi
   │
Tailscale
   │
Home Server
```

### Multi-User Environment

```text
User Devices
      │
      ▼
  Tailscale
      │
      ▼
 Backup Host
      │
      ├── tm-1
      ├── tm-2
      └── tm-3
```

---

## Credits

This project builds on the work of:

* Samba
* Docker
* Tailscale
* Apple Time Machine

Original project by Nick Zana.

---

## License

See the repository license for details.
