# FAST-LIO-LOCALIZATION

A simple localization framework that can re-localize in built maps based on [FAST-LIO](https://github.com/hku-mars/FAST_LIO). 


## News

- **2025-08-29:** Major update: Migrated to **Python 3.8+**, upgraded **Open3D** to v0.16.0, removed `ros_numpy`, and replaced `livox_ros_driver` with **livox_ros_driver2**. All build and run instructions are updated accordingly. If you are migrating from an older setup, please review the new environment and dependency requirements below.
- **2021-08-11:** Add **Open3D 0.7** support.
- **2021-08-09:** Migrate to **Open3D** for better performance.

## 1. Features
- Realtime 3D global localization in a pre-built point cloud map. 
  By fusing low-frequency global localization (about 0.5~0.2Hz), and high-frequency odometry from FAST-LIO, the entire system is computationally efficient.

<div align="center"><img src="doc/demo.GIF" width=90% /></div>

- Eliminate the accumulative error of the odometry.

<div align="center"><img src="doc/demo_accu.GIF" width=90% /></div>

- The initial localization can be provided either by rough manual estimation from RVIZ, or pose from another sensor/algorithm.

<!-- ![image](doc/real_experiment2.gif) -->
<!-- [![Watch the video](doc/real_exp_2.png)](https://youtu.be/2OvjGnxszf8) -->
<div align="center">
<img src="doc/demo_init.GIF" width=49.6% />
<img src="doc/demo_init_2.GIF" width = 49.6% >
</div>



## 2. Prerequisites

### Dependencies

- Python 3.8+
- [Open3D](http://www.open3d.org/docs/release/getting_started.html) (version 0.16.0 recommended)

```shell
pip install open3d==0.16.0
```


**Note:** The codebase now requires Python 3.8 or newer. `ros_numpy` is no longer required or used. All dependencies and code are updated for modern Open3D APIs.

If you encounter missing dependencies related to FAST-LIO, please refer to the [FAST-LIO documentation](https://github.com/hku-mars/FAST_LIO#1-prerequisites) for additional requirements that may be needed in your environment.


## 3. Build

Clone the repositories and build:

```
# Clone packages to your catkin workspace src directory
git clone https://github.com/Livox-SDK/livox_ros_driver2.git
git clone https://github.com/HViktorTsoi/FAST_LIO_LOCALIZATION.git
(cd FAST_LIO_LOCALIZATION && git submodule update --init)

# Build livox_ros_driver2
(cd livox_ros_driver2 && ./build.sh ROS1)

# Go back to workspace root and build everything
cd ../..
catkin_make
source devel/setup.bash
```

- **livox_ros_driver2** is now required (instead of livox_ros_driver). Make sure to update any references in `package.xml`, `CMakeLists.txt`, and source files if you are migrating from an older setup.
- If you want to use a custom build of PCL, add the following line to your `~/.bashrc`:
  ```export PCL_ROOT={CUSTOM_PCL_PATH}```


## 4. Run Localization
### 4.1 Sample Dataset

Demo rosbag in a large underground garage: 
[Google Drive](https://drive.google.com/file/d/15ZZAcz84mDxaWviwFPuALpkoeK-KAh-4/view?usp=sharing) | [Baidu Pan (Code: ne8d)](https://pan.baidu.com/s/1ceBiIAUqHa1vY3QjWpxwNA);

Corresponding map: [Google Drive](https://drive.google.com/file/d/1X_mhPlSCNj-1erp_DStCQZfkY7l4w7j8/view?usp=sharing) | [Baidu Pan (Code: kw6f)](https://pan.baidu.com/s/1Yw4vY3kEK8x2g-AsBi6VCw)

The map can be built using LIO-SAM or FAST-LIO-SLAM.

### 4.2 Run


1. First, please make sure you're using the **Python 3.8** environment;



2. Run localization, here we take Livox AVIA as an example:

```shell
roslaunch fast_lio_localization localization_avia.launch map:=/path/to/your/map.pcd
```

Please modify `/path/to/your/map.pcd` to your own map point cloud file path.

Wait for 3~5 seconds until the map cloud shows up in RVIZ;


3. If you are testing with the sample rosbag data:
```shell
rosbag play localization_test_scene_1.bag
```

Or if you are running realtime

```shell
roslaunch livox_ros_driver2 msg_mixed.launch
```
Please set the **publish_freq** in **livox_lidar_rviz.launch** to **10Hz**, to ensure there are enough points for global localization in a single scan. 
Support for higher frequency is coming soon.


4. Provide initial pose
```shell
rosrun fast_lio_localization publish_initial_pose.py 14.5 -7.5 0 -0.25 0 0 
```
The numerical value **14.5 -7.5 0 -0.25 0 0** denotes 6D pose **x y z yaw pitch roll** in map frame, 
which is a rough initial guess for **localization_test_scene_1.bag**. 

The initial guess can also be provided by the '2D Pose Estimate' Tool in RVIZ.

Note that, during the initialization stage, it's better to keep the robot still. Or if you play bags, firstly play the bag for about 0.5s, and then pause the bag until the initialization succeeds. 


## Related Works
1. [FAST-LIO](https://github.com/hku-mars/FAST_LIO): A computationally efficient and robust LiDAR-inertial odometry (LIO) package
2. [ikd-Tree](https://github.com/hku-mars/ikd-Tree): A state-of-art dynamic KD-Tree for 3D kNN search.
3. [FAST-LIO-SLAM](https://github.com/gisbi-kim/FAST_LIO_SLAM): The integration of FAST-LIO with [Scan-Context](https://github.com/irapkaist/scancontext) **loop closure** module.
4. [LIO-SAM_based_relocalization](https://github.com/Gaochao-hit/LIO-SAM_based_relocalization): A simple system that can relocalize a robot on a built map based on LIO-SAM.


## Acknowledgments
Thanks for the authors of [FAST-LIO](https://github.com/hku-mars/FAST_LIO) and [LIO-SAM_based_relocalization](https://github.com/Gaochao-hit/LIO-SAM_based_relocalization).

## TODO
1. Go over the timestamp issue of the published odometry and tf;
2. Using integrated points for global localization;
3. Fuse global localization with the state estimation of FAST-LIO, and smooth the localization trajectory; 
4. Updating...