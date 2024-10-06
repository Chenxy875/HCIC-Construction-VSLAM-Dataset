# HCIC-Construction-VSLAM-Dataset

**Developer**: Wenyu Li, Xinyu Chen, Yantao Yu

**Affiliation**: The Hong Kong University of Science and Technology

This is a description of our dataset paper: A real world visual slam dataset for indoor construction sites

<img src="figure/build1.gif" alt="build1" width="35%">  <img src="figure/floor1.gif" alt="floor1" width="35%">

## Download 

The dataset will be  available after the paper is accepted.

| Sequence            | Duration | Size  | Description                           | Link   |
| ------------------- | -------- | ----- | ------------------------------------- | ------ |
| Building A1, static | 85s      | 9.30G | Texture-less, multiple rooms          | Rosbag |
| Building B1, static | 78s      | 8.30G | Repetitive texture, looped trajectory | Rosbag |
| Floor14, static2    | 56s      | 7.32G | Over-exposure                         | Rosbag |
| Floor4,  static1    | 35s      | 4.56G | Glass, static workers                 | Rosbag |
| Floor4, static3     | 24s      | 3.08G | Texture-less, single room             | Rosbag |
| Floor1, dynamic     | 55s      | 5.43G | Motion blur, dynamic workers          | Rosbag |


- The Floor1 sequence is a segment of the raw data and does not contain tf messages. Transformations from other sequences can be used.



## Hardware

The dataset was collected using a RealSense L515 camera and an Ouster OS0-128 LiDAR. Further details about the sensors can be found in our paper.

<p align="center">
  <img src="figure/camera.png" alt="camera" height="250"> <img src="figure/lidar.png" alt="lidar" height="250">
</p>
<p align="center">RealSense L515 | Ouster OS0-128</p>

<p align="center">
  <img src="figure/data.png" alt="data" width="70%">
</p>
<p align="center">Sensor Data</p>


## Prerequisites
### Ubuntu and ROS

The dataset is recorded in rosbag format. We recommend using Ubuntu and ROS for optimal compatibility. The system has been tested on ROS Noetic and Ubuntu 20.04, but other versions should also work.

To use the provided lidar slam ground truth and visual slam benchmark, you need to install docker on your computer. If you are going to run them on your host machine, you might need to do some modification ( follow the original  repositories or instructions in our dockerfile).



### host machine(not recommend)

You need to check if these systems support your ubuntu or ros version. Melodic or Noetic should be a good choice. Make sure your could run popular slams on your computer and just directly use our provided rosbags in your own system.



### Docker(recommend)

If you are tired of struggling with the environments required by different repositories, you can have a try on our docker scripts.



Install docker engine and nvidia-docker(optional, but recommend), please follow the official docs.  **Do NOT install docker desktop.**

    https://docs.docker.com/engine/install/ubuntu/
    
    https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker



After installation, cd to the corresponding **docker** folder on your host machine, you will see three files. Change the name of docker image, docker container, path folder on your computer, etc.

```
├── ros_package_xxx/
│   ├── docker/
│   │   ├── build.sh
│   │   ├── Dockerfile
│   │   └── run.sh
│   ├── launch
│   ├── src
│   ├── ...
│   ├── CMakeLists.txt
│   └── package.xml
```



You can also run on a machine without nvidia display card. Just try :

```
# delete  "--gpus all " in the run .sh file.

docker run -it --name $container_name --gpus all \
```

```
# delete these lines in Dockerfile

ENV NVIDIA_VISIBLE_DEVICES ${NVIDIA_VISIBLE_DEVICES:-all}
ENV NVIDIA_DRIVER_CAPABILITIES ${NVIDIA_DRIVER_CAPABILITIES:+$NVIDIA_DRIVER_CAPABILITIES,}graphics
```



Compile docker image  on your host computer. It will take some time, depending on your internet.

```
bash build.sh
```



Start a new docker container. After that you can use the container(See docker official docs). You can use run.sh for multiple times. The old container will be instead with the new one.

```
bash run.sh
```



In our setting container,  /root is your working space. In the /root folder, catkin_ws is your ros workspace; dataset and downloads are connected to your host machine; start.sh is a starting script for compiling and preparation, you can change it at the end of the dockerfile.



To enable rviz in the container, enter on your host machine.

```
xhost +
```



### Sensor Drivers(optional)

If you want better support on the sensors, you may also want to install their ros drivers. To install on your host machine or docker container: 

**For RealSense:**

```
apt-get update && apt-get -y install apt-transport-https && \
    mkdir -p /etc/apt/keyrings && \
    curl -sSf https://librealsense.intel.com/Debian/librealsense.pgp | tee /etc/apt/keyrings/librealsense.pgp > /dev/null && \
    echo "deb [signed-by=/etc/apt/keyrings/librealsense.pgp] https://librealsense.intel.com/Debian/apt-repo `lsb_release -cs` main" | \
    tee /etc/apt/sources.list.d/librealsense.list 
```
```
apt-get update && apt-get install -y \
    librealsense2-dkms \
    librealsense2-utils \
    librealsense2-dev
```

```
apt-get update && apt install -y ros-melodic-realsense2-camera
```

**For Ouster:**

```
 apt-get update && apt-get install -y \
    ros-noetic-pcl-ros \
    ros-noetic-rviz \
    libspdlog-dev  
```

Download in your ros workspace.

```
 git clone --recurse-submodules https://github.com/ouster-lidar/ouster-ros.git
```



## Ground Truth

You are encouraged to use your own lidar slam as the ground truth based on our provided lidar data for higher accuracy evaluation. But here we also offer you an out of the box package configured in docker. It originates from [LIMO VELO](https://github.com/Huguet57/LIMO-Velo.git), a specific lidar SLAM modified from [FAST-LIO2](https://github.com/hku-mars/FAST_LIO.git).

In the docker container, modify the launchfile and then  run :

```
roslaunch limovelo ouster.launch
```




<p align="center">
  <a href="https://www.youtube.com/watch?v=UOO7NcGiOzY" title="YouTube video">YouTube</a> | 
  <a href="https://www.bilibili.com/video/BV1rx1RYCEm9/?share_source=copy_web&vd_source=06af5619f1080f2a613e3916fce2ae17" title="Bilibili video">Bilibili</a>
</p>

<p align="center">
  <img src="figure/lidar_test.png" alt="pic1">
</p>



## Benchmark

In our paper, we used VINS-RGBD as a baseline. However, we also provide both ORBSLAM3 and VINS-RGBD for benchmarking. They run in RGBD-Inertial mode.

Open the corresponding container and run:

```
roslaunch vins_estimator l515.launch 
```

or

```
roslaunch orbslam3 vio.launch
```



**For [vins rgbd](https://github.com/STAR-Center/VINS-RGBD) with rtabmap：**



<p align="center">
  <a href="https://www.youtube.com/watch?v=UBWpqnH1jf4" title="YouTube video">YouTube</a> | 
  <a href="https://www.bilibili.com/video/BV1r219YgEww/?share_source=copy_web&vd_source=06af5619f1080f2a613e3916fce2ae17" title="Bilibili video">Bilibili</a>
</p>

<p align="center">
  <img src="figure/camera_test.png" alt="pic1" width="70%">
</p>





**For [orbslam3 RGBD-I mode](https://github.com/xiefei2929/ORB_SLAM3-RGBD-Inertial) (it failed):**

<p align="center">
  <a href="https://www.youtube.com/watch?v=dwSeJD0URyo" title="YouTube video">YouTube</a> | 
  <a href="https://www.bilibili.com/video/BV1q919YvEWr/?vd_source=0cb5df4b054962e7162c762abbea8201" title="Bilibili video">Bilibili</a>
</p>

<p align="center">
  <img src="figure/orb.png" alt="pic1" width="70%">
</p>





## Evaluation

The localization results are saved in TUM format. You can use the [evo tool](https://github.com/MichaelGrupp/evo.git) for trajectory evaluation. Note that the extrinsic calibration between the LiDAR and the camera is not provided. For alignment in evo, use the --align option for automated alignment, or manually set a common starting point.


## Acknowledgments

We would like to thank the contributors of the referenced repositories. Please cite our paper if this dataset is useful for your work.
## Bibtex
Chen X, Yu Y. HLE-SLAM: SLAM for overexposed construction environment[C]//ISARC. Proceedings of the International Symposium on Automation and Robotics in Construction. IAARC Publications, 2023, 40: 585-588.

Chen X, Yu Y. An unsupervised low-light image enhancement method for improving V-SLAM localization in uneven low-light construction sites[J]. Automation in Construction, 2024, 162: 105404.
## Contact
If you have any problems with the dataset or software, please open an issue.

For academic or commercial collaboration, please contact the corresponding author [Prof. Yantao Yu](https://ce.hkust.edu.hk/people/yantao-yu-yuyantao).
