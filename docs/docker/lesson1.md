# Docker Architecture

1. **Docker Engine**
   - Uses Docker Daemon
   - Docker CLI
2. **Docker Client** $\rightarrow$ Creates the API (Application Programming Interface)
3. **Virtualization vs. Containerization**

---

# Steps to Set Up Docker in AWS

1. Go to AWS Management Console.
2. Search and click on **EC2 Instance**, then click **Launch Instance**.
3. Select an OS image (e.g., **Ubuntu**).
4. Select the instance type: **t2.micro** (Free-tier eligible: 1 vCPU, 1 GiB RAM).
5. Create a Key Pair:
   - Select algorithm: **RSA** or **ED25519** *(Recall the OpenSSH command)*
   - Private key format: **.pem** (Compatible with OpenSSH)
6. Configure Storage and launch the instance.

---

## Troubleshooting Permissions

After launching a new instance, connecting via SSH, and running a Docker command, you may encounter a `permission denied` error. This typically happens for two reasons:

1. **Docker Engine is not running.**
2. **Missing Group Permissions:** The logged-in user lacks permissions to access the Docker socket. To resolve this, add your current user to the `docker` group:

```bash
sudo usermod -aG docker $USER
newgrp docker # This is to refresh the groups
```

1. $USER -> Points to the current User
2. -aG -> Add group


