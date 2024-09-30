# Super-Simple-Alpine-Docker
Basic setup for reusable alpine Linux nodes on docker

# **Running Alpine Linux on macOS Using Docker**

This comprehensive guide provides step-by-step instructions to run Alpine Linux in a Docker container on your MacBook. It includes useful commands, configurations, and best practices to enhance your experience and ensure data integrity and security.

---

## **Table of Contents**

1. [Prerequisites](#1-prerequisites)
2. [Pulling the Alpine Linux Docker Image](#2-pulling-the-alpine-linux-docker-image)
3. [Running an Interactive Shell in the Alpine Container](#3-running-an-interactive-shell-in-the-alpine-container)
4. [Running the Container in Detached Mode](#4-running-the-container-in-detached-mode)
5. [Accessing the Container from Your Mac Terminal](#5-accessing-the-container-from-your-mac-terminal)
6. [Sharing Internet Connectivity](#6-sharing-internet-connectivity)
7. [Persistent Containers and Exiting](#7-persistent-containers-and-exiting)
8. [Assigning a Static IP to the Container](#8-assigning-a-static-ip-to-the-container)
9. [Allocating More Resources to Docker](#9-allocating-more-resources-to-docker)
10. [Properly Starting and Stopping the Docker Container](#10-properly-starting-and-stopping-the-docker-container)
11. [Completely Destroying the Docker Container and Image](#11-completely-destroying-the-docker-container-and-image)
12. [Security Considerations for Sensitive Work](#12-security-considerations-for-sensitive-work)
13. [Docker vs. VirtualBox](#13-docker-vs-virtualbox)
14. [Cheat Sheets and Helpful Commands](#14-cheat-sheets-and-helpful-commands)
15. [Networking in Docker](#15-networking-in-docker)
16. [Additional Tips](#16-additional-tips)
17. [Summary](#17-summary)

---

## **1. Prerequisites**

- **Docker Desktop for Mac**: Ensure you have Docker Desktop installed. Download it from [Docker Hub](https://www.docker.com/products/docker-desktop).
- **Basic Terminal Knowledge**: Familiarity with command-line operations.

---

## **2. Pulling the Alpine Linux Docker Image**

Open your macOS Terminal and pull the latest Alpine Linux image:

```bash
docker pull alpine
```

---

## **3. Running an Interactive Shell in the Alpine Container**

To start a new container and open an interactive shell:

```bash
docker run -it --name alpine_container alpine sh
```

- `-it`: Runs the container in interactive mode.
- `--name alpine_container`: Assigns a name to the container for easy reference.
- `alpine`: Specifies the image to use.
- `sh`: The command to execute inside the container.

You are now inside the Alpine Linux shell and can use `apk`, the Alpine package manager, to install packages.

**Example: Installing `curl`**

```bash
apk update
apk add curl
```

---

## **4. Running the Container in Detached Mode**

To keep the container running in the background even after closing the terminal:

```bash
docker run -d --name alpine_container alpine tail -f /dev/null
```

- `-d`: Runs the container in detached mode.
- `tail -f /dev/null`: Keeps the container alive by tailing `/dev/null`.

---

## **5. Accessing the Container from Your Mac Terminal**

To execute commands or access the shell of a running container:

- **Access the Shell**

  ```bash
  docker exec -it alpine_container sh
  ```

- **Run a Command**

  ```bash
  docker exec alpine_container <your_command_here>
  ```

  **Example:**

  ```bash
  docker exec alpine_container ls /
  ```

---

## **6. Sharing Internet Connectivity**

Docker containers share the host's network by default, allowing internet access without additional configuration.

**Testing Internet Connectivity Inside the Container**

```bash
docker exec alpine_container ping -c 4 google.com
```

If you receive ping responses, internet connectivity is working.

---

## **7. Persistent Containers and Exiting**

### **Entering Detached Containers**

To re-enter a detached container:

```bash
docker attach alpine_container
```

**Note:** Exiting with `Ctrl+C` will stop the container. To detach without stopping, use `Ctrl+P` followed by `Ctrl+Q`.

### **Exiting Without Stopping the Container**

- **Detach from Container Shell**

  Press:

  ```bash
  Ctrl+P, Ctrl+Q
  ```

This sequence detaches your terminal session from the container without stopping it.

---

## **8. Assigning a Static IP to the Container**

By default, Docker uses a bridge network. To assign a static IP:

### **Create a Custom Bridge Network**

```bash
docker network create --subnet=172.18.0.0/16 my_bridge
```

- `--subnet=172.18.0.0/16`: Defines the subnet for the network.
- `my_bridge`: Name of the custom network.

### **Run the Container with a Static IP**

```bash
docker run -d --name alpine_container --net my_bridge --ip 172.18.0.10 alpine tail -f /dev/null
```

Now, other devices on your network can access the container using `172.18.0.10` if network routing is properly configured.

**Note:** Exposing the container to your local network requires additional configuration and poses security risks. Proceed with caution.

---

## **9. Allocating More Resources to Docker**

Docker Desktop for Mac allows you to allocate resources via its settings:

1. **Open Docker Desktop Preferences**

   - Click on the Docker icon in the menu bar.
   - Select **Preferences**.

2. **Adjust Resources**

   - Navigate to the **Resources** tab.
   - Adjust **CPU**, **Memory**, and **Swap** according to your needs.
   - Click **Apply & Restart**.

---

## **10. Properly Starting and Stopping the Docker Container**

### **Starting the Docker Container**

- **Starting an Existing Container**

  ```bash
  docker start alpine_container
  ```

  This command starts a container that was previously stopped.

- **Running a New Container with Data Persistence**

  If you're creating a new container and need persistent data, consider mounting a volume:

  ```bash
  docker run -d --name alpine_container -v /path/on/mac:/path/in/container alpine tail -f /dev/null
  ```

  - `-v /path/on/mac:/path/in/container`: Mounts a directory from your Mac into the container, ensuring data persists outside the container's lifecycle.

### **Stopping the Docker Container Safely**

- **Graceful Shutdown**

  To stop the container gracefully, allowing running processes to exit properly:

  ```bash
  docker stop alpine_container
  ```

  This sends a `SIGTERM` signal, giving processes time to terminate.

- **Forcing a Stop (Not Recommended Unless Necessary)**

  ```bash
  docker kill alpine_container
  ```

  This sends a `SIGKILL` signal, immediately stopping all processes. Use this only if `docker stop` fails.

### **Exiting the Container Shell Without Stopping the Container**

When inside the container shell, avoid using `Ctrl+C` or `exit` if you want the container to keep running. Instead, detach:

- **Detach Without Stopping**

  Press:

  ```bash
  Ctrl+P, Ctrl+Q
  ```

  This detaches your terminal session from the container without stopping it.

### **Ensuring Data Persistence**

- **Use Volumes**

  Mounting volumes is crucial for data persistence:

  ```bash
  docker run -d --name alpine_container -v alpine_data:/data alpine tail -f /dev/null
  ```

  - `-v alpine_data:/data`: Creates a Docker named volume `alpine_data` mounted at `/data` inside the container.

- **Backing Up Data**

  Regularly back up important data from volumes:

  ```bash
  docker run --rm -v alpine_data:/data -v /backup/dir:/backup alpine tar cvf /backup/alpine_data_backup.tar /data
  ```

---

## **11. Completely Destroying the Docker Container and Image**

To remove all traces of the container and image, follow these steps:

### **Stop the Container**

First, ensure the container is stopped:

```bash
docker stop alpine_container
```

### **Remove the Container**

```bash
docker rm alpine_container
```

This deletes the container instance.

### **Remove Associated Volumes**

If you used named volumes, remove them:

- **List Volumes**

  ```bash
  docker volume ls
  ```

- **Remove a Volume**

  ```bash
  docker volume rm alpine_data
  ```

- **Remove All Unused Volumes**

  ```bash
  docker volume prune
  ```

  **Warning:** This removes all unused volumes. Ensure no other containers need them.

### **Remove the Docker Image**

```bash
docker rmi alpine
```

If the image is being used by other containers, you'll need to remove those containers first.

### **Remove Build Cache and Unused Data**

```bash
docker system prune --all --volumes
```

- `--all`: Removes all unused images, not just dangling ones.
- `--volumes`: Removes all unused volumes.

**Warning:** This is a destructive action. Ensure you don't need any of the resources being removed.

### **Verify Removal**

- **List Containers**

  ```bash
  docker ps -a
  ```

  You should not see `alpine_container`.

- **List Images**

  ```bash
  docker images
  ```

  The `alpine` image should no longer be listed.

- **List Volumes**

  ```bash
  docker volume ls
  ```

  Ensure that `alpine_data` or any associated volumes are gone.

### **Secure Deletion**

If you need to ensure data is irrecoverable:

- **Overwrite Data Before Deletion**

  Inside the container or on the host, overwrite sensitive data before deleting.

  **Example in Container:**

  ```bash
  dd if=/dev/zero of=/data/sensitive_file bs=1M count=10
  ```

- **Use Secure Deletion Tools on Host**

  Tools like `shred` or `srm` can securely delete files on your Mac.

  ```bash
  brew install coreutils  # If not already installed
  gshred -u /path/to/file
  ```

  **Note:** Secure deletion on SSDs may not be entirely effective due to wear-leveling algorithms.

---

## **12. Security Considerations for Sensitive Work**

Using Docker containers can be secure for sensitive tasks if best practices are followed.

### **Benefits**

- **Isolation**

  Containers provide process isolation, reducing the risk of interference with the host system.

- **Controlled Environment**

  You can define the exact environment needed, minimizing unexpected behavior.

### **Security Best Practices**

- **Use Minimal Base Images**

  Alpine is a minimal image, reducing the attack surface.

- **Run as Non-Root User**

  By default, containers run as root, which can be risky.

  **Create a Non-Root User in the Container:**

  ```bash
  # Inside Dockerfile or Container
  adduser -D -g '' user
  su user
  ```

  Or modify your Dockerfile to switch users.

- **Limit Container Capabilities**

  Use the `--cap-drop` flag to drop unnecessary Linux capabilities.

  ```bash
  docker run --cap-drop ALL --cap-add NET_BIND_SERVICE alpine
  ```

- **Use Read-Only Filesystems**

  ```bash
  docker run --read-only alpine
  ```

- **Network Security**

  - **Disable Unnecessary Ports**

    Avoid exposing ports unless required.

  - **Use Private Networks**

    Keep sensitive containers on isolated Docker networks.

- **Regular Updates**

  Keep your Docker Engine and images updated to patch security vulnerabilities.

### **Potential Risks**

- **Shared Kernel**

  Docker containers share the host OS kernel. A vulnerability in the kernel or misconfigured container can lead to security breaches.

- **Data Persistence**

  Data stored inside containers without volumes may be lost or corrupted.

### **Additional Measures**

- **Use Docker Bench for Security**

  Docker provides a script to check for common security issues.

  ```bash
  docker run -it --net host --pid host --cap-add audit_control \
    -v /var/lib:/var/lib -v /var/run/docker.sock:/var/run/docker.sock \
    docker/docker-bench-security
  ```

- **Secrets Management**

  Do not store sensitive information in images or environment variables. Use Docker secrets or an external secrets manager.

- **Monitor and Log Activity**

  Implement logging to monitor container activity.

- **Consider Using Docker in Rootless Mode**

  Running Docker as a non-root user can enhance security.

### **When to Use Virtual Machines Instead**

For highly sensitive work, consider using a virtual machine (VM):

- **Full Isolation**

  VMs provide hardware-level isolation.

- **Separate Kernel**

  Each VM has its own kernel, reducing risk from kernel vulnerabilities.

- **Enhanced Security Features**

  VMs can leverage security features like secure boot and encrypted disks.

---

## **13. Docker vs. VirtualBox**

- **Docker**

  - Uses containerization to run applications in isolated environments.
  - Shares the host's kernel, making it lightweight.
  - Ideal for running individual applications or services.

- **VirtualBox**

  - Uses virtualization to run entire operating systems.
  - Emulates hardware, consuming more resources.
  - Suitable for full OS environments with GUIs.

---

## **14. Cheat Sheets and Helpful Commands**

### **Docker Commands**

- **List Running Containers**

  ```bash
  docker ps
  ```

- **List All Containers**

  ```bash
  docker ps -a
  ```

- **Start a Stopped Container**

  ```bash
  docker start alpine_container
  ```

- **Stop a Running Container**

  ```bash
  docker stop alpine_container
  ```

- **Restart a Container**

  ```bash
  docker restart alpine_container
  ```

- **Remove a Container**

  ```bash
  docker rm alpine_container
  ```

- **Remove an Image**

  ```bash
  docker rmi alpine
  ```

- **Inspect a Container**

  ```bash
  docker inspect alpine_container
  ```

- **Show Container Logs**

  ```bash
  docker logs alpine_container
  ```

- **Prune Unused Resources**

  ```bash
  docker system prune
  ```

### **Alpine Linux Commands**

- **Update Package Index**

  ```bash
  apk update
  ```

- **Upgrade Installed Packages**

  ```bash
  apk upgrade
  ```

- **Install a Package**

  ```bash
  apk add <package_name>
  ```

- **Remove a Package**

  ```bash
  apk del <package_name>
  ```

- **Search for a Package**

  ```bash
  apk search <keyword>
  ```

### **File Sharing Between Host and Container**

- **Mount a Volume**

  ```bash
  docker run -it -v /path/on/mac:/path/in/container alpine sh
  ```

  Replace `/path/on/mac` with the directory on your Mac and `/path/in/container` with the mount point inside the container.

### **Networking Commands**

- **List Docker Networks**

  ```bash
  docker network ls
  ```

- **Inspect a Network**

  ```bash
  docker network inspect my_bridge
  ```

- **Connect a Container to a Network**

  ```bash
  docker network connect my_bridge alpine_container
  ```

- **Disconnect a Container from a Network**

  ```bash
  docker network disconnect my_bridge alpine_container
  ```

### **Exposing Ports**

To make a container's service accessible, map its port to the host:

```bash
docker run -d -p 8080:80 alpine
```

- `-p 8080:80`: Maps port `80` in the container to port `8080` on the host.

---

## **15. Networking in Docker**

### **Understanding Docker Networks**

- **Bridge Network**

  Default network type; containers on the same bridge network can communicate.

- **Host Network**

  Container shares the host's networking stack.

- **Overlay Network**

  Used in Docker Swarm for multi-host networking.

### **Custom Bridge Network**

Creating a custom bridge network allows for better control:

```bash
docker network create my_bridge
```

Connect containers to this network to enable communication.

### **DNS in Docker**

Docker provides internal DNS resolution. Containers can resolve each other by name if on the same network.

---

## **16. Additional Tips**

### **Leverage macOS's UNIX Compatibility**

macOS has a UNIX-based terminal. You can install GNU utilities using Homebrew:

- **Install Homebrew**

  ```bash
  /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
  ```

- **Install Core Utilities**

  ```bash
  brew install coreutils
  ```

### **Creating Docker Aliases**

Add aliases to your shell configuration (`~/.bashrc` or `~/.zshrc`) for frequent commands:

```bash
alias dps='docker ps'
alias dstart='docker start alpine_container'
alias dstop='docker stop alpine_container'
alias dexec='docker exec -it alpine_container sh'
```

### **Cleaning Up Docker Resources**

- **Remove Unused Containers**

  ```bash
  docker container prune
  ```

- **Remove Unused Images**

  ```bash
  docker image prune
  ```

- **Remove All Unused Data**

  ```bash
  docker system prune
  ```

**Use these commands with caution, as they will remove data.**

---

## **17. Summary**

- **Running Alpine Linux in Docker**

  - Pull the image and run it interactively.
  - Use detached mode to keep the container running in the background.

- **Networking**

  - Containers have internet access by default.
  - Assign static IPs using custom bridge networks.

- **Resource Allocation**

  - Adjust CPU and memory in Docker Desktop preferences.

- **Properly Starting and Stopping Containers**

  - Use `docker start` and `docker stop` for safe operations.
  - Avoid data corruption by ensuring processes terminate gracefully.

- **Completely Destroying Containers and Images**

  - Remove containers with `docker rm`.
  - Remove images with `docker rmi`.
  - Clean up volumes and system resources with `docker system prune`.

- **Security Considerations**

  - Docker can be secure if best practices are followed.
  - For highly sensitive tasks, evaluate whether Docker's isolation is sufficient.
  - Consider using VMs for enhanced security.

- **Docker vs. VirtualBox**

  - Docker is lightweight and shares the host kernel.
  - VirtualBox virtualizes hardware for full OS environments.

- **Useful Commands**

  - Manage containers with `docker run`, `docker exec`, `docker stop`, etc.
  - Install packages in Alpine using `apk`.

- **Additional Tips**

  - Use aliases for efficiency.
  - Keep your system clean with prune commands.
