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

   then clear to clear the terminal
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
   sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   sudo chmod +x /usr/local/bin/docker-compose

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


## Step 9: Navigate to Model Management
1. In the Open WebUI interface, click the **Settings** icon (gear) in the top-right or left sidebar.
2. Select **Models** from the settings menu to open the model management page.

## Step 10: Add Gemma3:1b
1. In the **Models** section, locate the input field for adding a new model.
2. Enter the model name exactly as: `gemma3:1b`.
3. Click the **Download** button (usually an arrow or plus icon) next to the input field.
4. A progress bar or status indicator will appear, showing the download progress. The `Gemma3:1b` model is approximately 2GB, so download time depends on your internet speed.
5. Once downloaded, the model will appear in the list of available models.

## Step 11: Add Gemma3:4b
1. Repeat the process for `Gemma3:4b`.
2. In the same **Models** section, enter the model name: `gemma3:4b`.
3. Click the **Download** button.
4. The `Gemma3:4b` model is approximately 5GB, so ensure sufficient disk space and wait for the download to complete.
5. The model will be added to the available models list once downloaded.

## Step 5: Verify Model Availability
1. Return to the main chat interface in Open WebUI.
2. Click the model dropdown menu (usually at the top of the chat window).
3. Confirm that `gemma3:1b` and `gemma3:4b` appear in the list.
4. Select one of the models and send a test prompt (e.g., “What is 2 + 2?”) to ensure it responds correctly.



## Step 12: Troubleshooting
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


# Installing LiteLLM on Raspberry Pi 5 with Existing Open WebUI and API Integrations via Admin UI

This guide provides instructions for installing LiteLLM on a Raspberry Pi 5, configuring it using the LiteLLM Admin UI where possible, connecting it to an existing Open WebUI installation, integrating with Grok, Anthropic, and OpenAI APIs, and testing to ensure all models work correctly. The project will be organized in a `litellm` folder.

## Introduction

LiteLLM is an AI proxy that simplifies managing multiple AI models from different providers through a unified API endpoint. Paired with an existing Open WebUI setup, it enables a user-friendly interface for interacting with AI models. This guide leverages the LiteLLM Admin UI to streamline configuration.

## Prerequisites

- **Hardware**: Raspberry Pi 5 (8GB RAM recommended)
- **Operating System**: Raspberry Pi OS Bookworm (64-bit)
- **Internet Connection**: Required for downloading packages and Docker images
- **Open WebUI**: Already installed and running (e.g., on port 3000)
- **API Keys**:
  - Grok API key (from xAI, see https://x.ai/api)
  - Anthropic API key
  - OpenAI API key
- Basic familiarity with terminal commands

## Step 1: Update the Raspberry Pi and Install Docker

1. Update your Raspberry Pi:
   ```bash
   sudo apt update
   sudo apt full-upgrade -y
   ```

2. Install Docker if not already installed:
   ```bash
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   ```

3. Add your user to the Docker group to run Docker without `sudo`:
   ```bash
   sudo usermod -aG docker $USER
   ```

4. Log out and back in, or run:
   ```bash
   newgrp docker
   ```

5. Verify Docker installation:
   ```bash
   docker --version
   ```

## Step 2: Set Up the Project Directory

Create a directory for the LiteLLM project:
```bash
mkdir ~/litellm && cd ~/litellm
```

## Step 3: Set Up Docker Compose for LiteLLM

Create a `docker-compose.yml` file to run LiteLLM:
```bash
nano docker-compose.yml
```
Add the following:
```yaml
version: '3.9'
services:
  litellm:
    image: ghcr.io/berriai/litellm:main-latest
    ports:
      - "4000:4000"
    env_file:
      - ./.env
    command: --port 4000
```
Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

## Step 4: Configure LiteLLM Environment

1. Create a `.env` file to store the LiteLLM master key, salt key, and Admin UI credentials:
   ```bash
   nano .env
   ```
   Add the following, replacing `your_secure_password` with a strong password:
   ```
   LITELLM_MASTER_KEY=sk-X7Kp9mL2Qw
   LITELLM_SALT_KEY=sk-Z9vR4jP8Ys
   ```
   Save and exit.

2. Start the LiteLLM service:
   ```bash
   docker-compose up -d
   ```

## Step 5: Configure LiteLLM via Admin UI

1. Open a browser and navigate to `http://<Raspberry-Pi-IP>:4000/ui` (e.g., `http://192.168.1.100:4000/ui`). Find your IP with:
   ```bash
   hostname -I
   ```

2. Log in with:
   - Username: `admin`
   - Password: `your_secure_password` (from `.env`)

3. **Add API Keys**:
   - Go to the **Keys** or **Settings** section in the Admin UI.
   - Add the following API keys (replace placeholders with your actual keys):
     - **Grok**: Name it `GROK_API_KEY`, value `your_grok_api_key`
     - **Anthropic**: Name it `ANTHROPIC_API_KEY`, value `your_anthropic_api_key`
     - **OpenAI**: Name it `OPENAI_API_KEY`, value `your_openai_api_key`
   - Save the keys.

4. **Configure Models**:
   - Navigate to the **Models** section.
   - Add the following models (adjust model names based on provider documentation):
     - **Grok Model**:
       - Model Name: `grok-model`
       - Provider: `grok`
       - Model: `grok/model-name` (replace with actual Grok model, e.g., `grok/created_by_xai`)
       - API Key: Select `GROK_API_KEY`
     - **Claude-3-Sonnet**:
       - Model Name: `claude-3-sonnet`
       - Provider: `anthropic`
       - Model: `claude-3-sonnet-20240229`
       - API Key: Select `ANTHROPIC_API_KEY`
     - **GPT-4o**:
       - Model Name: `gpt-4o`
       - Provider: `openai`
       - Model: `gpt-4o`
       - API Key: Select `OPENAI_API_KEY`
   - Save each model configuration.

5. **Create a Virtual Key** for Open WebUI:
   - Go to the **Keys** section.
   - Generate a new virtual key (e.g., `sk-webui-5678`).
   - Copy this key for use in Open WebUI.

## Step 6: Test LiteLLM

Verify LiteLLM is working by sending a test request:
```bash
curl -X POST http://localhost:4000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer sk-X7Kp9mL2Qw" \
  -d '{
    "model": "gpt-4o",
    "messages": [{"role": "user", "content": "Hello"}]
  }'
```
You should receive a JSON response with a greeting. Repeat for `grok-model` and `claude-3-sonnet` to ensure all models are accessible.

## Step 7: Connect Open WebUI to LiteLLM

1. Open a browser and go to `http://<Raspberry-Pi-IP>:3000` (where Open WebUI is running).
2. Log in with your Open WebUI admin account.
3. Go to **Admin Panel > Settings > Connections > OpenAI API**.
4. Enable the OpenAI API connection.
5. Set:
   - **Base URL**: `http://<Raspberry-Pi-IP>:4000/v1` (e.g., `http://192.168.1.100:4000/v1`)
   - **API Key**: `sk-webui-5678` (the virtual key from Step 5)
6. Save the settings.

## Step 8: Test Open WebUI

1. In Open WebUI, go to the chat interface.
2. Check the model dropdown; you should see `grok-model`, `claude-3-sonnet`, and `gpt-4o`.
3. For each model:
   - Select the model.
   - Send a test prompt (e.g., “What is the capital of France?”).
   - Verify the model responds correctly (e.g., “The capital of France is Paris.”).
4. If any model fails, check the LiteLLM logs:
   ```bash
   docker logs litellm-litellm-1
   ```

## Step 9: Configure Firewall for Network Access

To allow other devices on the network to access the LiteLLM Admin UI (Open WebUI is assumed to be already accessible):

1. Install `ufw` if not present:
   ```bash
   sudo apt install ufw -y
   ```

2. Allow port 4000 (LiteLLM):
   ```bash
   sudo ufw allow 4000/tcp
   ```

3. Enable the firewall if not already enabled:
   ```bash
   sudo ufw enable
   ```

4. Verify firewall status:
   ```bash
   sudo ufw status
   ```

## Step 10: Keeping LiteLLM Updated

To update LiteLLM to the latest version:
1. Navigate to the project directory:
   ```bash
   cd ~/litellm
   ```
2. Pull the latest image and restart:
   ```bash
   docker-compose pull
   docker-compose up -d
   ```

## Troubleshooting

- **Admin UI not accessible**: Ensure LiteLLM is running (`docker ps`) and port 4000 is open (`sudo lsof -i :4000`).
- **Models not appearing in Open WebUI**: Verify the Base URL and API key in Open WebUI settings, and check model configurations in the LiteLLM Admin UI.
- **API errors**: Confirm API keys are correct in the Admin UI and models are supported (see LiteLLM documentation or provider model lists).
- **Performance issues**: Ensure no other heavy processes are running on the Raspberry Pi 5, as API-based models rely on network latency and provider performance.

## Notes

- **Security**: The LiteLLM Admin UI is exposed on the network. For home use, this is generally safe, but for office environments, consider using a VPN or additional authentication. Avoid exposing port 4000 to the public internet without securing it.
- **Grok API**: Model names may vary. Check https://x.ai/api for the latest Grok model details.
- **LiteLLM Admin UI**: Some advanced configurations may require editing `config.yaml`. Refer to [LiteLLM documentation](https://docs.litellm.ai/) if needed.

## References

- [LiteLLM Documentation](https://docs.litellm.ai/)
- [Open WebUI Documentation](https://docs.openwebui.com/)
- [xAI API](https://x.ai/api)
- [Anthropic API](https://docs.anthropic.com/)
- [OpenAI API](https://platform.openai.com/docs/)
