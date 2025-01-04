# Minecraft Server (Bedrock Edition)

This project involves setting up a Minecraft Bedrock Edition server using Ubuntu, Docker, and Tailscale. The system is hosted on an HP ProDesk 600 G4 Mini Desktop. The server is currently set up for local access, but there are plans to bring it to PlayIt.gg for public access in the future.

## Technologies Used
- **Ubuntu**: Operating system (you could use Windows instead)
- **Docker**: Containerization for the Minecraft server
- **Tailscale**: VPN for secure network access

## System Details
- **Hardware**: HP ProDesk 600 G4 Mini Desktop (i5 8600T)

## Future Plans
- Move the server to PlayIt.gg (reverse proxy service) for public access

## Steps to Recreate the Project

1. **Set Up Ubuntu**
   - Install Ubuntu on your system. [Ubuntu Installation Guide](https://ubuntu.com/tutorials/install-ubuntu-desktop)
   
2. **Install Docker**
   - Run the following commands to install Docker on Ubuntu:
     ```bash
     sudo apt update
     sudo apt install docker.io
     ```

3. **Create Docker Container for Minecraft**
   - Pull the Minecraft Bedrock Edition Docker image:
     ```bash
     docker pull itzg/minecraft-bedrock-server
     ```
   - Create and run the Docker container:
     ```bash
     docker run -d -p 19132:19132/udp --name minecraft-bedrock itzg/minecraft-bedrock-server
     ```

4. **Set Up Tailscale**
   - Install Tailscale on Ubuntu:
     ```bash
     curl -fsSL https://tailscale.com/install.sh | sh
     sudo tailscale up
     ```
   - Connect to your Tailscale network for secure access.

5. **Test the Server**
   - Connect to your Minecraft server locally or via Tailscale using the IP address provided.

## Screenshots/Images
- ![Server Screenshot](path/to/screenshot.png)

## Future Updates
- Transition to PlayIt.gg for public server access.
