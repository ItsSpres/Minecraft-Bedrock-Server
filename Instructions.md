# Homelab Setup Guide: Portainer, Tailscale, and Minecraft Bedrock Server

This document provides setup instructions for Portainer, Tailscale, and Minecraft Bedrock Server, ensuring they are properly configured and accessible in your homelab.

---

## **1. Portainer Setup**
Portainer is a web-based interface for managing Docker containers.

### **Step 1: Install Portainer**
1. Create a Docker volume for Portainer data:
   ```bash
   docker volume create portainer_data
   ```

2. Run the Portainer container:
   ```bash
   docker run -d -p 8000:8000 -p 9000:9000 \
     --name portainer \
     --restart=unless-stopped \
     -v /var/run/docker.sock:/var/run/docker.sock \
     -v portainer_data:/data \
     portainer/portainer-ce
   ```

3. Access the Portainer web interface:
   - Open a browser and go to: `http://<your-server-ip>:9000`
   - Create an admin account and log in.

### **Step 2: Persistent Storage**
Store Portainer data in `/srv/docker-data/portainer` (optional).
```bash
sudo mkdir -p /srv/docker-data/portainer
sudo chown -R $USER:$USER /srv/docker-data/portainer
```
Update the `docker run` command to:
```bash
docker run -d -p 8000:8000 -p 9000:9000 \
  --name portainer \
  --restart=unless-stopped \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /srv/docker-data/portainer:/data \
  portainer/portainer-ce
```

---

## **2. Tailscale Setup**
Tailscale is a VPN service for securely accessing your network.

### **Step 1: Install Tailscale**
1. Install Tailscale:
   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   ```

2. Authenticate Tailscale:
   ```bash
   sudo tailscale up
   ```
   Follow the URL provided in the terminal to log in to your Tailscale account.

3. Verify your Tailscale IP:
   ```bash
   tailscale ip
   ```
   Note the IP (e.g., `100.x.x.x`) for use with the Minecraft server.

### **Step 2: Enable Tailscale at Boot**
1. Enable Tailscale to start on boot:
   ```bash
   sudo systemctl enable tailscaled
   ```
2. Start the Tailscale service:
   ```bash
   sudo systemctl start tailscaled
   ```

---

## **3. Minecraft Bedrock Server Setup**

### **Step 1: Install Minecraft Bedrock Server**
1. Pull the Minecraft Bedrock Server Docker image:
   ```bash
   docker pull itzg/minecraft-bedrock-server
   ```

2. Run the server container:
   ```bash
   docker run -d --name minecraft-bedrock-server \
     -p 19132:19132/udp \
     -v /srv/docker-data/minecraft:/data \
     -e EULA=TRUE \
     --restart unless-stopped \
     itzg/minecraft-bedrock-server
   ```

### **Step 2: Replace World with Backup**
1. Stop the container:
   ```bash
   docker stop minecraft-bedrock-server
   ```

2. Replace the world folder:
   ```bash
   cp -r /path/to/backup/Bedrock\ level /srv/docker-data/minecraft/worlds/
   ```

3. Set permissions:
   ```bash
   sudo chown -R $USER:$USER /srv/docker-data/minecraft/worlds/Bedrock\ level
   sudo chmod -R 755 /srv/docker-data/minecraft/worlds/Bedrock\ level
   ```

4. Restart the container:
   ```bash
   docker start minecraft-bedrock-server
   ```

5. Verify the server logs:
   ```bash
   docker logs minecraft-bedrock-server
   ```

### **Step 3: Connect to the Server**
1. Use your **Tailscale IP** or local IP.
2. Connect via Minecraft Bedrock Edition:
   - **Address**: `<Tailscale IP>`
   - **Port**: `19132`

---

## **4. Centralized Storage Setup**

### **Step 1: Create a Centralized Folder**
1. Create a main folder:
   ```bash
   mkdir -p /srv/docker-data
   ```

2. Create subfolders for each service:
   ```bash
   mkdir -p /srv/docker-data/{portainer,minecraft,resume-matcher}
   ```

3. Set permissions:
   ```bash
   sudo chown -R $USER:$USER /srv/docker-data
   sudo chmod -R 755 /srv/docker-data
   ```

### **Step 2: Share the Folder on the Network**
#### **Option 1: Share via Samba**
1. Install Samba:
   ```bash
   sudo apt install samba -y
   ```
2. Configure Samba:
   ```bash
   sudo nano /etc/samba/smb.conf
   ```
   Add the following at the bottom:
   ```ini
   [DockerData]
   path = /srv/docker-data
   browseable = yes
   writable = yes
   guest ok = yes
   read only = no
   ```
3. Restart Samba:
   ```bash
   sudo systemctl restart smbd
   ```
4. Access the share:
   - **Windows**: `\\<Ubuntu-IP>\DockerData`
   - **Linux/Mac**: `smb://<Ubuntu-IP>/DockerData`

---

## **5. Stop All Services**

To stop Portainer, Tailscale, and Minecraft server simultaneously, create a script:
1. Create a script file:
   ```bash
   nano stop_services.sh
   ```
2. Add the following:
   ```bash
   #!/bin/bash
   docker stop minecraft-bedrock-server
   sudo systemctl stop tailscaled
   docker stop portainer
   ```
3. Make it executable:
   ```bash
   chmod +x stop_services.sh
   ```
4. Run the script:
   ```bash
   ./stop_services.sh
   
