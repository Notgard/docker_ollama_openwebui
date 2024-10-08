# Docker + Ollama + OpenWebUI Installation instructions

## Make sure install scripts can be executed
```bash
chmod +x docker_install.sh && chmod +x container_toolkit_install.sh
```

## Install docker
```bash
./docker_install.sh
```

## Install nvidia-drivers
```bash
sudo ubuntu-drivers install
```

## Install nvidia-container-toolkit
```bash
./container_toolkit_install.sh
```

# Need to reboot required to test proper driver installation (driver nvidia-550)

### Note:
System should boot properly and nvidia functions available

### Test
If the driver installation worked, the following command should show GPU information
```bash
nvidia-smi
```

## Pull and run ollama with GPU acceleration
```bash
sudo docker run -d --gpus=all -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

## Check if container is running properly
```bash
sudo docker ps
```

## Pull and run openwebui with Nvidia GPU support
```bash
sudo docker run -d -p 3000:8080 --gpus all --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:cuda
```

## Check if container(s) is(are) running properly
```bash
sudo docker ps
```

## Open your web browser and head to : http://localhost:3000
OpenWebUI should appear in your browser if everything worked.
To use OpenWebUI, creating a user and logging in is required : https://github.com/open-webui/open-webui/discussions/491
