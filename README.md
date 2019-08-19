# SparkVIO_ROS

ROS Wrapper for [SparkVIO](https://github.mit.edu/SPARK/VIO).

# 1. Installation

## A. Prerequisities

### i. ROS

Install ROS by following [our reference](./docs/ros_installation.md), or the official [ROS website](https://www.ros.org/install/).

### ii. SparkVIO's dependencies

Follow installation instructions in [SparkVIO](https://github.mit.edu/SPARK/VIO/blob/master/docs/sparkvio_installation.md).
Make sure you install **SparkVIO's dependencies**: GTSAM, OpenCV, OpenGV.

SparkVIO itself can be installed by cloning the **[SparkVIO catkin wrapper](https://github.mit.edu/SPARK/spark_vio_catkin)** in your catkin workspace, so you can spare installing SparkVIO from source (its dependencies must be installed anyway).

## B. SparkVIO ROS wrapper Installation

If you have the above prerequisities and [SparkVIO](https://github.mit.edu/SPARK/VIO) installed and built, installation of the SparkVIO ROS wrapper should be:

```bash
# Setup catkin workspace
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws/
catkin init

# Add workspace to bashrc.
echo 'source ~/catkin_ws/devel/setup.bash' >> ~/.bashrc

# Clone repo
cd ~/catkin_ws/src
git clone git@github.mit.edu:SPARK/spark_vio_ros.git

# Install dependencies from rosinstall file using wstool
wstool init
wstool merge spark_vio_ros/install/spark_vio.rosinstall
wstool update
```

Clone SparkVIO catkin wrapper (**only if you haven't installed SparkVIO already**).
```bash
# Clone SparkVIO catkin wrapper, useful if you don't want to build spark vio from source.
git clone git@github.mit.edu:SPARK/spark_vio_catkin.git
```

Finally, compile:

```bash
# Compile code
catkin build

# Refresh workspace
source ~/.bashrc
```

# 2. Usage
Download a [Euroc](https://projects.asl.ethz.ch/datasets/doku.php?id=kmavvisualinertialdatasets) rosbag: for example [V1_01_easy](http://robotics.ethz.ch/~asl-datasets/ijrr_euroc_mav_dataset/vicon_room1/V1_01_easy/V1_01_easy.bag).

## Online
  - In one terminal, launch the SparkVIO ROS wrapper:
  ```bash
  roslaunch spark_vio_ros spark_vio_ros_euroc.launch
  ```
  - In another terminal, launch the downloaded Euroc rosbag: 
  ```bash
  rosbag play /PATH/TO/EUROC_ROSBAG --clock
  ```

## Offline
  In this mode, the provided rosbag will be first parsed and then sent to the VIO for processing.
  This is particularly useful when debugging to avoid potential ROS networking issues.
  - To run, launch the SparkVIO ROS wrapper with the `online` parameter set to `false` and specify the rosbag's path:
  ```bash
  roslaunch spark_vio_ros spark_vio_ros_euroc.launch online:=false rosbag_path:="PATH/TO/ROSBAG"
  ```

# Hardware use
## RealSense D435i (Infrared)

Why do we use the infrared cameras on the D435i?
The infrared cameras offer the option to run the SparkVIO stereo version on monochrome global shutter cameras, which are generally better suited for visual tracking.

### Setup

1. Download and install the [Intel RealSense SDK](https://github.com/IntelRealSense/librealsense/blob/development/doc/distribution_linux.md)

2. Download and install the [Intel RealSense ROS wrapper](https://github.com/IntelRealSense/realsense-ros)

3. Adapt the RealSense ROS wrapper to publish a single interpolated IMU message [(see nodelet xml)](https://github.com/IntelRealSense/realsense-ros/blob/c2448916218ccfe49b0d642563493cb4e9bdcc3b/realsense2_camera/launch/includes/nodelet.launch.xml#L82)

4. Make sure to properly cover the infrared projector on the RealSense (this otherwise affects the quality of the infrared image with dots)

5. Collect calibration bagfiles for camera intrinsics and extrinsics [(see instructions)](https://www.youtube.com/watch?v=puNXsnrYWTY&app=desktop)

6. Calibrate camera intrinsics and extrinsics using [Kalibr](https://github.com/ethz-asl/kalibr)

7. Create configuration files for SparkVIO ROS wrapper using [Kalibr2SparkVIO-pinhole-radtan](https://github.mit.edu/SPARK/VIO/blob/feature/parallelization/jpl/kalibr/kalibr2sparkvio_stereo_pinhole-radtan.py)

8. Create/adapt your own specific launch file, similar to [example RealSense IR](https://github.mit.edu/SPARK/spark_vio_ros/blob/jpl/launch/spark_vio_ros_realsense_IR.launch)

### Testing

1. Launch RealSense camera using ```roslaunch realsense2_camera [name of your launch file]```

2. Visualize image stream using ```rosrun image_view image_view image:=[name of camera topic]```

3. Launch SparkVIO ROS wrapper using ```roslaunch spark_vio_ros [name of your launch file]```

4. Visualize trajectory with RVIZ using ```rviz```, [(see example config)](https://github.mit.edu/SPARK/spark_vio_ros/blob/jpl/viz/visualize_sparkvio.rviz)

5. Visualize state and statistics using ```rqt_multiplot```, [(see example config)](https://github.mit.edu/SPARK/spark_vio_ros/blob/jpl/viz/rqt_multiplot_state.xml)

It is important to remember that when launching the VIO, the camera should be standing still and upward (camera fov forward looking).

## MyntEye S

### Setup

1. Download and install the [MyntEye SDK and ROS wrapper](https://github.com/slightech/MYNT-EYE-S-SDK)

2. Collect calibration bagfiles for camera intrinsics and extrinsics [(see instructions)](https://www.youtube.com/watch?v=puNXsnrYWTY&app=desktop)

3. Calibrate camera intrinsics and extrinsics using [Kalibr](https://github.com/ethz-asl/kalibr), recommended model is: ```pinhole-equi``` [(see OpenCV documentation)](https://docs.opencv.org/3.3.1/db/d58/group__calib3d__fisheye.html)

4. Create configuration files for SparkVIO ROS wrapper using [Kalibr2SparkVIO-pinhole-equi](https://github.mit.edu/SPARK/VIO/blob/feature/parallelization/jpl/kalibr/kalibr2sparkvio_stereo_pinhole-equi.py) or [[Kalibr2SparkVIO-pinhole-radtan](https://github.mit.edu/SPARK/VIO/blob/feature/parallelization/jpl/kalibr/kalibr2sparkvio_stereo_pinhole-equi.py)]

5. Create/adapt your own specific launch file, similar to [example MyntEye S](https://github.mit.edu/SPARK/spark_vio_ros/blob/jpl/launch/spark_vio_ros_mynteye.launch)

### Testing

1. Launch MyntEye camera using ```roslaunch mynt_eye_ros_wrapper [name of your launch file]```

2. Visualize image stream using ```rosrun image_view image_view image:=[name of camera topic]```

3. Launch SparkVIO ROS wrapper using ```roslaunch spark_vio_ros [name of your launch file]``` (example, see below)

4. Visualize trajectory with RVIZ using ```rviz```, [(see example config)](https://github.mit.edu/SPARK/spark_vio_ros/blob/jpl/viz/visualize_sparkvio.rviz)

5. Visualize state and statistics using ```rqt_multiplot```, [(see example config)](https://github.mit.edu/SPARK/spark_vio_ros/blob/jpl/viz/rqt_multiplot_state.xml)

#### Example

For the MyntEyes used in SubT: (online)
```
roslaunch spark_vio_ros spark_vio_ros_mynteye.launch camera:=JPL distortion:=equidistant
```
Options for camera are ```MIT``` and ```JPL```. Options for distortion are ```equidistant``` and ```radtan```.

Same goes for use offline, using the ```spark_vio_ros_mynteye_offline.launch``` file and an additional ```data``` argument with path to bagfile.

# BSD License
SparkVIO ROS wrapper is open source under the BSD license, see the [LICENSE.BSD](./LICENSE.BSD) file.
