# Setting Up Open WebUI on Raspberry Pi 5 with Ollama and Network Access

This guide provides step-by-step instructions to install and configure Open WebUI on a Raspberry Pi 5, connect it to an existing Ollama installation, and make it accessible to all devices on your home or office network. These instructions assume you have already set up Ollama on your Raspberry Pi 5 and are running Raspberry Pi OS Bookworm (64-bit).

## Prerequisites
- **Raspberry Pi 5** (8GB RAM recommended for better performance)
- **Raspberry Pi OS Bookworm** (64-bit, latest version)
- **Ollama** installed and running with at least one language model (e.g., TinyLlama, Phi3, or Llama3)
- **Docker** installed on the Raspberry Pi
- Internet connection for initial setup
- Basic familiarity with terminal commands

## Step 1: Update the Raspberry Pi
Ensure your Raspberry Pi is up-to-date to avoid compatibility issues.

1. Open a terminal on your Raspberry Pi.
2. Run the following commands:
   ```bash
   sudo apt update
   sudo apt full-upgrade -y
   ```

## Step 2: Install Docker (if not already installed)
Docker is required to run Open WebUI. If Docker is not installed, follow these steps:

1. Install Docker using the official script:
   ```bash
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   ```
2. Add your user to the Docker group to run Docker without sudo:
   ```bash
   sudo usermod -aG docker $USER
   ```
3. Log out and back in, or run `newgrp docker` to apply the group change.
4. Verify Docker installation:
   ```bash
   docker --version
   ```

## Step 3: Configure Ollama to Listen on All Network Interfaces
By default, Ollama listens only on `localhost`. To allow Open WebUI and other devices on the network to connect, configure Ollama to listen on all IP addresses.

1. Edit the Ollama service file:
   ```bash
   sudo systemctl edit ollama.service
   ```
2. In the editor, add the following lines under `[Service]` to set the environment variable:
   ```
   [Service]
   Environment="OLLAMA_HOST=0.0.0.0:11434"
   ```
3. Save and exit the editor.
4. Reload the systemd daemon:
   ```bash
   sudo systemctl daemon-reload
   ```
5. Restart the Ollama service:
   ```bash
   sudo systemctl restart ollama
   ```
6. Verify Ollama is running and accessible:
   ```bash
   curl http://localhost:11434
   ```
   You should see a response indicating Ollama is running.

## Step 4: Install Open WebUI with Docker
Open WebUI will be installed using Docker for simplicity and ease of updates.

1. Create a directory for Open WebUI:
   ```bash
   mkdir ~/openwebui
   cd ~/openwebui
   ```
2. Create a Docker Compose file to define the Open WebUI service:
   ```bash
   nano docker-compose.yml
   ```
3. Add the following content to `docker-compose.yml`:
   ```yaml
   version: '3.9'
   services:
     open-webui:
       image: ghcr.io/open-webui/open-webui:main
       container_name: open-webui
       volumes:
         - open-webui-data:/app/backend/data
       ports:
         - "3000:8080"
       environment:
         - OLLAMA_BASE_URL=http://host.docker.internal:11434
       extra_hosts:
         - "host.docker.internal:host-gateway"
       restart: unless-stopped
   volumes:
     open-webui-data:
   ```
   **Note**: The `OLLAMA_BASE_URL` assumes Ollama is running on the same Raspberry Pi. If Ollama is on a different device, replace `host.docker.internal` with the IP address of the Ollama host (e.g., `http://192.168.1.100:11434`).
4. Save and exit the editor (`Ctrl+O`, `Enter`, `Ctrl+X`).
5. Start the Open WebUI container:
   ```bash
   docker-compose up -d
   ```
6. Verify the container is running:
   ```bash
   docker ps
   ```

## Step 5: Configure Firewall for Network Access
To allow other devices on the network to access Open WebUI, configure the Raspberry Pi’s firewall to allow incoming traffic on port 3000.

1. If using `ufw` (Uncomplicated Firewall), install it if not already present:
   ```bash
   sudo apt install ufw -y
   ```
2. Allow incoming traffic on port 3000:
   ```bash
   sudo ufw allow 3000/tcp
   ```
3. Enable the firewall if not already enabled:
   ```bash
   sudo ufw enable
   ```
4. Check the firewall status:
   ```bash
   sudo ufw status
   ```

## Step 6: Access Open WebUI
1. On the Raspberry Pi, open a web browser and navigate to `http://localhost:3000`.
2. From another device on the same network, find the Raspberry Pi’s IP address:
   ```bash
   hostname -I
   ```
   Then, in a browser on the other device, navigate to `http://<Raspberry-Pi-IP>:3000` (e.g., `http://192.168.1.100:3000`).
3. The first user to sign up will have admin privileges. Click “Sign Up” and create an account.

## Step 7: Connect Open WebUI to Ollama
Open WebUI should automatically detect Ollama if configured correctly.

1. In the Open WebUI interface, go to the chat interface.
2. Select a model from the dropdown (e.g., TinyLlama, Phi3, or Llama3) that you previously installed with Ollama.
3. If no models appear, go to **Settings > Admin Settings > Connections > Ollama > Manage** and ensure the connection is set to `http://<Raspberry-Pi-IP>:11434` or `http://host.docker.internal:11434`.
4. Test the connection by typing a prompt (e.g., “What is the capital of France?”) and pressing Enter.

## Step 8: Manage Models (Optional)
You can manage Ollama models directly through Open WebUI.

1. In Open WebUI, go to **Settings > Models**.
2. To download a new model, enter the model name (e.g., `gemma:2b`) and click the download button.
3. Wait for the model to download, then select it from the chat interface.

## Step 9: Troubleshooting
- **Open WebUI cannot connect to Ollama**:
  - Ensure Ollama is running: `sudo systemctl status ollama`.
  - Verify the `OLLAMA_BASE_URL` in the `docker-compose.yml` file.
  - Check that Ollama is listening on `0.0.0.0:11434` using `netstat -tuln | grep 11434`.
- **Cannot access Open WebUI from other devices**:
  - Confirm the Raspberry Pi’s IP address and port 3000 are accessible.
  - Check firewall settings: `sudo ufw status`.
  - Ensure no other services are using port 3000: `sudo lsof -i :3000`.
- **Performance issues**:
  - Use lightweight models like TinyLlama or Phi3 for better performance on the Raspberry Pi 5.
  - Ensure sufficient RAM (8GB recommended).

## Step 10: Keeping Open WebUI Updated
To update Open WebUI to the latest version:

1. Navigate to the Open WebUI directory:
   ```bash
   cd ~/openwebui
   ```
2. Pull the latest image and restart the container:
   ```bash
   docker-compose pull
   docker-compose up -d
   ```

## Notes
- **Security**: For home use, this setup is generally safe. For office environments, consider adding authentication or using a VPN for secure access. Avoid exposing port 3000 to the public internet without proper security measures.
- **Performance**: The Raspberry Pi 5 is capable but limited. Larger models like Llama3 may be slow. Stick to lightweight models for better responsiveness.
- **Model Selection**: Visit [Ollama’s model library](https://ollama.com/library) to choose models suitable for your needs and hardware.

## References
- [Pi My Life Up: Self-hosting an AI Chatbot with Open WebUI on the Raspberry Pi](https://pimylifeup.com/raspberry-pi-open-webui/)[](https://pimylifeup.com/raspberry-pi-open-webui/)
- [Open WebUI Documentation](https://docs.openwebui.com/)[](https://docs.openwebui.com/)
- [Ollama on Raspberry Pi](https://pimylifeup.com/raspberry-pi-ollama/)[](https://pimylifeup.com/raspberry-pi-ollama/)
