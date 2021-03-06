# SAS Event Stream Processing Docker Container with GPU acceleration

This repository provides build instructions to create your own SAS Event Stream Processing Docker Container with GPU acceleration.<br>
Included is an installation of [OpenCV 4.4.0](https://github.com/Mentos05/OpenCV-x64-gpu) for Computer Vision tasks.

### Overview
SAS Event Stream Processing already offers the possibility to be installed as a Docker container. So, why does this repository exist?
Looking at the official [documentation](https://go.documentation.sas.com/?docsetId=dplyesp0phy0lax&docsetTarget=p1rcii2jo7dt9yn1qr1upedc0y3w.htm&docsetVersion=6.2&locale=en) you will notice that the Docker image does not support GPU acceleration.

However GPU acceleration is needed or at least recommended if you want to work with deep learning.
For this reason I decided to share my work of creating GPU accelerated for SAS Event Stream Processing.

Please note:
This repository is privately owned by me. Don't expect any official support for the work provided here.

### Requirements
* Valid SAS Event Stream Processing license file (tested with SAS Event Stream Processing 6.2)
* System with NVIDIA GPU (tested with RTX2070, RTX3000 and V100)
* Linux OS (tested with Ubuntu 18.04 and Centos 7)
* NVIDIA Driver (tested with version 430/450)
* [NVIDIA Docker](https://github.com/NVIDIA/nvidia-docker)

### Container Setup
1. Clone this repository<br>
```
git clone https://github.com/Mentos05/SAS_ESP_GPU.git
cd SAS_ESP_JETSON_GPU
```
2. Copy your license file into the repository folder (usually named: SAS_Viya_deployment_data.zip)
```
cp /path/to/your/SAS_Viya_deployment_data.zip .
```
3. (Optional) Add additional SAS Event Stream Processing adapters/plugins to the appropriate folders in esp_addons.
4. Run buildContainer.sh. The script will download all required files and build the container.
```
bash buildcontainer.sh
```

### Customize your Container Setup (Optional)
The buildContainer.sh script accepts user variables.<br>
Append them to buildContainer.sh with --variable=value<br>
Example:<br>
```
bash buildContainer.sh --sas_deployment_data=mydeploymentdata.zip
```

| Variable | Description | Default |
| ------ | ------ | ------ |
| sas_deployment_data | Name of your SAS deployment data file | SAS_Viya_deployment_data.zip |
| sas_mirrormanager_url | URL for SAS Mirror Manager | https://support.sas.com/installation/viya/35/sas-mirror-manager/lax/mirrormgr-linux.tgz |
| sas_mirrorextensions_url | URL for SAS Mirror Manager Extension | https://support.sas.com/installation/viya/35/sas-edge-extension/sas-edge-extension.tgz |
| sas_software_repository | Name of the SAS software repository | sas-espedge-125-aarch64_ubuntu_linux_16-apt |
| sas_package_location | Location of SAS packages | espedge_repos/sas-espedge-106-x64_redhat_linux_6-yum |
| sas_plugins_location | Location of ESP Plugins | esp_addons/esp_plugins |
| sas_adapters_location | Location of ESP Adapters | esp_addons/additional_adapters |
| python_req | Location of requirements file for Python | requirements.txt |
| opencv_package_location | Location of OpenCV RPM packages | OpenCV-x64-gpu/opencv-centos7-x64-rpm |
| container_name | Docker container name | esp_gpu |
| container_tag | Docker container tag | 6_2 |
| container_build | Build container | YES |
| container_push | Push container to repository | NO |

### Run Container
Simply use docker run and attach your gpus:
```
docker run -it --net=host --gpus all esp_gpu:6_2
```
In many cases you want to extend your run call with additional variables to configure the container.

| Variable | Description | Default |
| ------ | ------ | ------ |
| ESP_PORT | ESP -port option | 9900 |
| ESP_PUBSUB_PORT | ESP -pubsub option | 31416 |
| ESP_LOGLEVEL | ESP -loglevel option | error |
| ESP_MAS_THREADS | MAS Threads | 1 |
| MAS_PYLOG_LEVEL | MAS logging level | ERROR |
| MAS_PYLOG_FILE | MAS logfile location | /opt/maspylog.txt |
| JUPYTERLAB_PORT | JupyterLab port | 8080 |
| JUPYTERLAB_NBDIR | JupyterLab notebook directory | /data/notebooks/ |

Example: `docker run -it --net=host -e ESP_PORT=12345 --gpus all esp_gpu:6_2` will run the ESP server on port 12345.

You should see something like this:<br>
![cv](images/running_container.png "cv")

### Access Jupyter Lab
Open the following URL in your browser:<br>
http://localhost:8080

### Whats next?
Connect to your container, e.g. via [SAS ESPPy](https://github.com/sassoftware/python-esppy).<br>
Your container is running on port 9900 by default.
```
import esppy
esp = esppy.ESP(hostname='http://localhost:9900')
```

### Jupyter Lab (Python Environment)
Some of the Python packages installed are:<br>

| Package | Description |
| ------ | ------ |
| SAS SWAT | [SAS SWAT](https://github.com/sassoftware/python-swat) |
| SAS DLPy | [SAS DLPy](https://github.com/sassoftware/python-dlpy) | 
| SAS ESPPy | [SAS ESPPy](https://github.com/sassoftware/python-esppy) |
| OpenCV | [OpenCV](https://github.com/opencv/opencv) |
| TensorFlow | [TensorFlow](https://github.com/tensorflow/tensorflow) |

For a full list, please look at the requirements.txt folder.
When building your own container, you can adjust this file as you like to ex/include Python packages.

### Share ressources with your container
If you want to share ressources with your container, e.g. a webcam, you can do so by adapting your docker run command.<br>
To share devices, e.g. your webcam, use:
```
docker run --device=/dev/video0:/dev/video0 --net=host esp_gpu:6_2
```
To share a folder, e.g. with additional data like models, projects, etc. use:
```
docker run -v folder-on-host:folder-on-container --net=host esp_gpu:6_2
```

Example: For my needs I usually start my container with the following command to share my local notebooks, my webcam, host networking interface and to allow GUI applications (e.g. Opencv).<br>
```
docker run -it --privileged=true --net=host --ipc=host \
           -v /home/michael/Development/github.com/:/data/notebooks \
           --device=/dev/video0:/dev/video0 \
           --gpus all -e DISPLAY=$DISPLAY \
           -v /tmp/.X11-unix:/tmp/.X11-unix -v /var/run/dbus:/var/run/dbus \
           esp_gpu:6_2
```

### Run GUI applications inside your Docker container
I am using OpenCV very often to display the scored images from ESP. To allow OpenCV to access your hosts display you'll have to allow access to your X server.
To do this simply type `xhost +` on your host system.
Additionally you'll have to provide some information to your container by adding the following statements to your run-command:<br>
```
-e DISPLAY=$DISPLAY
-v /tmp/.X11-unix:/tmp/.X11-unix
```

### Verify/Monitor GPU Usage
While you should notice a significant performance improvement while training/scoring your deep learning models you can also monitor GPU usage by using:
```
watch -n 1 nvidia-smi
```
nvidia-smi should show a esp process.
![nvidia-smi monitoring](images/nvidia-smi.png "nvidia-smi monitoring")<br>
Make sure you run this command on your host, not inside the container.

### Private Repository
Please Note: This is my private repository and not an official SAS repository.<br>
If you are looking for official SAS repositories, please go to:
* [SAS Software Repository](https://github.com/sassoftware/)
* [SAS Scripting Wrapper for Analytics Transfer (SWAT)](https://github.com/sassoftware/python-swat)
* [SAS Viya Deep Learning API for Python](https://github.com/sassoftware/python-dlpy)
* [SAS Event Stream Processing Python Interface](https://github.com/sassoftware/python-esppy)

### Contact
If you like to discuss how deep learning can be applied to your problems, you're of course free to contact me.<br>

| Channel | Adress |
| ------ | ------ |
| Email Work | michael.gorkow@sas.com |
| Email Private | michaelgorkow@gmail.com |
| LinkedIn | [LinkedIn Profile](https://www.linkedin.com/in/michael-gorkow-08353678/) |
| Twitter | [Twitter Profile](https://twitter.com/GorkowMichael) |
