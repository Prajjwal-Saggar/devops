# SSH, SSHD & SCP Reference Guide

## 1. What is SSH?

**SSH (Secure Shell)** is a network protocol used to securely connect to and manage remote servers over an encrypted connection.

```bash
# Connect to a remote server
ssh username@remote_host_ip
```

---

## 2. What is SSHD?

`sshd` (Secure Shell Daemon) is the background service running on the remote server. It listens for incoming SSH connection requests (default Port 22), authenticates credentials, and manages active client sessions.

### Common `sshd` Commands (Run on Server)

```bash
# Check service status
sudo systemctl status sshd

# Start, stop, or restart the service
sudo systemctl start sshd
sudo systemctl stop sshd
sudo systemctl restart sshd

# Enable service to start automatically on system boot
sudo systemctl enable sshd

# Edit SSH server configuration settings
sudo nano /etc/ssh/sshd_config
```

---

## 3. File Transfer with SCP

`scp` (Secure Copy Protocol) uses SSH to transfer files and directories between a local client and a remote server.

> **Important:** Run all `scp` commands from your local machine's terminal, not inside an active SSH session.

### Upload (Local → Remote Server)

```bash
# Upload a single file
scp /path/to/local/file.txt username@server_ip:/path/to/remote/

# Upload an entire directory (-r)
scp -r /path/to/local/folder username@server_ip:/path/to/remote/
```

### Download (Remote Server → Local)

```bash
# Download a single file
scp username@server_ip:/path/to/remote/file.txt /path/to/local/

# Download an entire directory (-r)
scp -r username@server_ip:/path/to/remote/folder /path/to/local/
```
