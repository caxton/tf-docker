## Tensorflow Docker Image (Refer from https://github.com/floydhub/dl-docker)

## Specs
This is what you get out of the box when you create a container with the provided image/Dockerfile:
* Ubuntu 16.04
* [CUDA 8.0](https://developer.nvidia.com/cuda-toolkit) (GPU version only)
* [cuDNN v6](https://developer.nvidia.com/cudnn) (GPU version only)
* [Tensorflow](https://www.tensorflow.org/)

## Setup
### Prerequisites
1. Install Docker following the installation guide for your platform: [https://docs.docker.com/engine/installation/](https://docs.docker.com/engine/installation/)

2. **GPU Version Only**: Install Nvidia drivers on your machine either from [Nvidia](http://www.nvidia.com/Download/index.aspx?lang=en-us) directly or follow the instructions [here](https://github.com/saiprashanths/dl-setup#nvidia-drivers). Note that you _don't_ have to install CUDA or cuDNN. These are included in the Docker container.

3. **GPU Version Only**: Install nvidia-docker: [https://github.com/NVIDIA/nvidia-docker](https://github.com/NVIDIA/nvidia-docker), following the instructions [here](https://github.com/NVIDIA/nvidia-docker/wiki/Installation). This will install a replacement for the docker CLI. It takes care of setting up the Nvidia host driver environment inside the Docker containers and a few other things.

### Obtaining the Docker image
You have 2 options to obtain the Docker image
#### Option 1: Download the Docker image from Docker Hub

**CPU Version**
```bash
docker pull caxton/tf-docker:cpu
```

**GPU Version**
```bash
docker pull caxton/tf-docker:gpu
```

#### Option 2: Build the Docker image locally
Alternatively, you can build the images locally. Also, since the GPU version is not available in Docker Hub at the moment, you'll have to follow this if you want to GPU version. Note that this will take an hour or two depending on your machine since it compiles a few libraries from scratch.

```bash
git clone https://github.com/caxton/tf-docker.git
cd dl-docker
```	

**CPU Version**
```bash
docker build -t caxton/tf-docker:cpu -f Dockerfile.cpu .
```

**GPU Version**
```bash
docker build -t caxton/tf-docker:gpu -f Dockerfile.gpu .
```
This will build a Docker image named `dl-docker` and tagged either `cpu` or `gpu` depending on the tag your specify. Also note that the appropriate `Dockerfile.<architecture>` has to be used.

## Running the Docker image as a Container
Once we've built the image, we have all the frameworks we need installed in it. We can now spin up one or more containers using this image, and you should be ready to [go deeper](http://imgur.com/gallery/BvuWRxq)

**CPU Version**
```bash
docker run -it -p 8888:8888 -p 6006:6006 -v /sharedfolder:/root/sharedfolder caxton/tf-docker:cpu bash
```
	
**GPU Version**
```bash
nvidia-docker run -it -p 8888:8888 -p 6006:6006 -v /sharedfolder:/root/sharedfolder floydhub/dl-docker:gpu bash
```
Note the use of `nvidia-docker` rather than just `docker`

| Parameter      | Explanation |
|----------------|-------------|
|`-it`             | This creates an interactive terminal you can use to iteract with your container |
|`-p 8888:8888 -p 6006:6006`    | This exposes the ports inside the container so they can be accessed from the host. The format is `-p <host-port>:<container-port>`. The default iPython Notebook runs on port 8888 and Tensorboard on 6006 |
|`-v /sharedfolder:/root/sharedfolder/` | This shares the folder `/sharedfolder` on your host machine to `/root/sharedfolder/` inside your container. Any data written to this folder by the container will be persistent. You can modify this to anything of the format `-v /local/shared/folder:/shared/folder/in/container/`. See [Docker container persistence](#docker-container-persistence)
|`floydhub/dl-docker:cpu`   | This the image that you want to run. The format is `image:tag`. In our case, we use the image `dl-docker` and tag `gpu` or `cpu` to spin up the appropriate image |
|`bash`       | This provides the default command when the container is started. Even if this was not provided, bash is the default command and just starts a Bash session. You can modify this to be whatever you'd like to be executed when your container starts. For example, you can execute `docker run -it -p 8888:8888 -p 6006:6006 floydhub/dl-docker:cpu jupyter notebook`. This will execute the command `jupyter notebook` and starts your Jupyter Notebook for you when the container starts

## Some common scenarios
### Jupyter Notebooks
The container comes pre-installed with iPython and iTorch Notebooks, and you can use these to work with the deep learning frameworks. If you spin up the docker container with `docker-run -p <host-port>:<container-port>` (as shown above in the [instructions](#running-the-docker-image-as-a-container)), you will have access to these ports on your host and can access them at `http://127.0.0.1:<host-port>`. The default iPython notebook uses port 8888 and Tensorboard uses port 6006. Since we expose both these ports when we run the container, we can access them both from the localhost.

However, you still need to start the Notebook inside the container to be able to access it from the host. You can either do this from the container terminal by executing `jupyter notebook` or you can pass this command in directly while spinning up your container using the `docker run -it -p 8888:8888 -p 6006:6006 caxton/tf-docker:cpu jupyter notebook` CLI. The Jupyter Notebook has both Python (for TensorFlow, Caffe, Theano, Keras, Lasagne) and iTorch (for Torch) kernels.

Note: If you are setting the notebook on Windows, you will need to first determine the IP address of your Docker container. This command on the Docker command-line provides the IP address
```bash
docker-machine ip default
> <IP-address>
```
```default``` is the name of the container provided by default to the container you will spin. 
On obtaining the IP-address, run the docker as per the [instructions](#running-the-docker-image-as-a-container) provided and start the Jupyter notebook as [described above](#jupyter-notebooks). Then accessing ```http://<IP-address>:<host-port>``` on your host's browser should show you the notebook.

* [Tensorflow Docker](https://github.com/tensorflow/tensorflow/tree/master/tensorflow/tools/docker)
