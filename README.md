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
## Warning ! The following command reboots your system, save any unsaved documents beforehand
```bash
sudo reboot
```

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

### Note:
The Nvidia GPU support for WebUI isn't necessary in most cases (confusing) :
https://github.com/open-webui/open-webui/discussions/2167

## Check if container(s) is(are) running properly
```bash
sudo docker ps
```

## Open your web browser and head to : http://localhost:3000
OpenWebUI should appear in your browser if everything worked.
To use OpenWebUI, creating a user and logging in is required : https://github.com/open-webui/open-webui/discussions/491

## Force disable logging
OpenWebUI can only disable logging into an account for fresh installs or by removing already existing users.  
The easiest way for docker is to clean the docker emulation memory (volume) if you already started one with the following command :

```bash
sudo docker rm --volumes open-webui # 'open-webui' being the name of the container
```

Now you simply need to run the container with **WEBUI_AUTH** set to **False** again like the following :  
```bash
sudo docker run -d -p 3000:8080 --gpus all --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data -e WEBUI_AUTH=False --name open-webui --restart always ghcr.io/open-webui/open-webui:cuda
```

---

## Running Jupyterlab server from docker container (With GPU acceleration but without CUDA compilation)

The following command runs a simple Jupyterlab server with acess to the GPU from the inside. It does not include additional libraries for AI or packages for advanced GPU features with CUDA. For proper utilization of the GPU for AI workloads, consider reading the next part with **full GPU access**.

```bash
PWD="$(pwd)" && sudo docker run --rm --gpus all -p 8889:8888 --workdir /work --mount type=bind,source=$PWD,target=/work quay.io/jupyter/base-notebook start-notebook.py --NotebookApp.token='my-token'
```

where "my-token" is the name of the token used to access the server from the following URL :   
http://localhost:8889/lab?token=my-token or simply http://localhost:8889

The command will bind your current directory when running the docker command into the /work directory inside the container.   
Meaning inside your container, your files will be in /work. You can change this behavior with the PWD variable (by default the current directory).  
For example, **PWD="$(pwd)/source"** will put the work from the source subfolder into the Jupyter container

## Running JupyterLab server from docker container with full GPU access
With the previous installations on your local machine, you should be able to access your GPU (nvidia-smi) directly and from your docker container.  
The previous Jupyterlab instructions make GPU accelerated available as they have access to your GPU, but with some limitations. AI libraries like **pytorch** or others may not be able to fully utilize your GPU because it lacks compilation capibilities (**nvcc**).  
  
To circonmvent this, run the following command :  
```bash
PWD="$(pwd)" && PASSWORD="test" && sudo docker run --rm --gpus all -d -it -p 8889:8888 -v jupyterlab:/home/jovyan/work --workdir /work --mount type=bind,source=$PWD,target=/work -e JUPYTER_TOKEN=$PASSWORD -e NB_UID=$(id -u) -e NB_GID=$(id -g) --env-file ./env.list --user root --name gpu_jupyter_server cschranz/gpu-jupyter:v1.7_cuda-12.2_ubuntu-22.04_slim && sudo docker logs gpu_jupyter_server
```
  
or (if you don't want the env.list file constantly)
  
```bash
PWD="$(pwd)" && PASSWORD="test" && sudo docker run --rm --gpus all -d -it -p 8889:8888 -v jupyterlab:/home/jovyan/work --workdir /work --mount type=bind,source=$PWD,target=/work -e JUPYTER_TOKEN=$PASSWORD -e NB_UID=$(id -u) -e NB_GID=$(id -g) -e GRANT_SUDO=yes -e JUPYTER_ENABLE_lAB=yes --user root --name gpu_jupyter_server cschranz/gpu-jupyter:v1.7_cuda-12.2_ubuntu-22.04_slim && sudo docker logs gpu_jupyter_server
```

This will run a Jupyterlab server that includes not only full GPU support with CUDA enabled, as well as some minimal AI libraries. The default password to use when prompted to access the Jupyterlab server here is simply "test", you can change this to whichever password by modifying the PASSWORD variable.  
After running the command, head to the following URL in your browser to access Jupyterlab :  
http://localhost:8889
