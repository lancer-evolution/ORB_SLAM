# ORB-SLAM Monocular

本家の[ORB_SLAM](https://github.com/raulmur/ORB_SLAM)では`rosbuild`でビルドしており、自分の環境では動かなかったので`catkin`でビルドできるように修正した．

##### 実行環境
ubuntu 14.04
ROS indigo
OpenCV 2.4.10

# 1. License

ORB-SLAM is released under a [GPLv3 license]
If you use ORB-SLAM in an academic work, please cite:

    @article{murAcceptedTRO2015,
      title={{ORB-SLAM}: a Versatile and Accurate Monocular {SLAM} System},
      author={Mur-Artal, Ra\'ul, Montiel, J. M. M. and Tard\'os, Juan D.},
      journal={IEEE Transactions on Robotics},
      volume={31},
      number={5},
      pages={1147--1163},
      doi = {10.1109/TRO.2015.2463671},
      year={2015}
     }


# 2. Prerequisites (dependencies)

## 2.1 Boost

	sudo apt-get install libboost-all-dev

## 2.2 OpenCV 2.4.10
Dowload and install instructions can be found at: http://opencv.org/

## 2.3 g2o (included in Thirdparty) with Eigen3
We use a modified version of g2o (see original at https://github.com/RainerKuemmerle/g2o) to perform optimizations.
In order to compile g2o you will need to have installed Eigen3 (at least 3.1.0).
	
	sudo apt-get install libeigen3-dev

## 2.5 DBoW2 (included in Thirdparty) 
We make use of some components of the DBoW2 and DLib library (see original at https://github.com/dorian3d/DBoW2) for place recognition and feature matching. There are no additional dependencies to compile DBoW2.

# 3. Installation

1. preparations
		git clone git@github.com:lancer-evolution/ORB_SLAM.git -b catkin
		echo "export ROS_PACKAGE_PATH=${ROS_PACKAGE_PATH}:PATH_TO_PARENT_OF_ORB_SLAM" >> ~/.bashrc
		source ~/.bashrc
`PATH_TO_PARENT_OF_ORB_SLAM`は各々の`ORB_SLAM`パッケージへのパス

2. Build g2o. Go into `ORB_SLAM/Thirdparty/g2o/` and execute:

		mkdir build
		cd build
		cmake .. -DCMAKE_BUILD_TYPE=Release
		make

	*Tip: To achieve the best performance in your computer, set your favorite compilation flags in line 61 and 62 of* `Thirdparty/g2o/CMakeLists.txt`(by default -03 -march=native)

5. Build DBoW2. Go into `ORB_SLAM/Thirdparty/DBoW2/`and execute:

		mkdir build
		cd build
		cmake ..
		make
	*Tip: Set your favorite compilation flags in line 4 and 5 of* `Thirdparty/DBoW2/CMakeLists.txt`(by default -03 -march=native)

4. Build ORB_SLAM.
		cd ~/catkin_ws
		catkin_make

# 4. Usage

**See section 5 to run the Example Sequence**.

1. Launch ORB-SLAM from the terminal (`roscore` should have been already executed):

		rosrun ORB_SLAM ORB_SLAM Data/ORBvoc.txt Data/Settings.yaml
ここで、引数はORB vocabulary と settings fileのパスである.  
  `ORBvoc.txt`は`ORB_SLAM/Data/ORBvoc.txt.tar.gz`を事前に展開しておく. 

2. 特徴点の抽出状況は`/ORB_SLAM/Frame`で配信されており，それを`image_view`で見ることができる:

		rosrun image_view image_view image:=/ORB_SLAM/Frame _autosize:=true

3. マップは`/ORB_SLAM/Map`で配信されており, 現在のカメラポーズとワールド座標系は`/tf` の `/ORB_SLAM/Camera` と `/ORB_SLAM/World` で配信されている.  マップを見るために `rviz` を起動する:

		rosrun rviz rviz -d Data/rviz.rviz

4. ORB_SLAM は `/camera/image_raw`からカメラ情報を取得している. よって、このトピックでカメラの情報をPublishしてやればいい．  
もし、連続的なカメラ画像のみがあるのであれば、https://github.com/raulmur/BagFromImages を使えば、画像群からbagファイルを作ってくれるらしい.


**Tip: 以上の`ORB_SLAM`, `image_view` and `rviz` を一括で起動するlaunchファイルは以下のとおりである**:

	roslaunch ORB_SLAM ExampleGroovyOrNewer.launch


# 5. Example Sequence
We provide the settings and the rosbag of an example sequence in our lab. In this sequence you will see a loop closure and two relocalisation from a big viewpoint change.

1. Download the rosbag file:  
	http://webdiis.unizar.es/~raulmur/orbslam/downloads/Example.bag.tar.gz. 

	Alternative link: https://drive.google.com/file/d/0B8Qa2__-sGYgRmozQ21oRHhUZWM/view?usp=sharing

	Uncompress the file.

2. Launch ORB_SLAM with the settings for the example sequence. You should have already uncompressed the vocabulary file (`/Data/ORBvoc.txt.tar.gz`)

  *in ROS Fuerte*:

	  roslaunch ExampleFuerte.launch

	*in ROS Groovy or newer versions*:

	  roslaunch ExampleGroovyHydro.launch

3. Once the ORB vocabulary has been loaded, play the rosbag (press space to start):

		rosbag play --pause Example.bag


# 6. The Settings File

ORB_SLAM reads the camera calibration and setting parameters from a YAML file. We provide an example in `Data/Settings.yaml`, where you will find all parameters and their description. We use the camera calibration model of OpenCV.

Please make sure you write and call your own settings file for your camera (copy the example file and modify the calibration)

# 7. Failure Modes

You should expect to achieve good results in sequences similar to those in which we show results in our paper [1], in terms of camera movement and texture in the environment. In general our Monocular SLAM solution is expected to have a bad time in the following situations:
- No translation at system initialization (or too much rotation).
- Pure rotations in exploration.
- Low texture environments.
- Many (or big) moving objects, especially if they move slowly.

The system is able to initialize from planar and non-planar scenes. In the case of planar scenes, depending on the camera movement relative to the plane, it is possible that the system refuses to initialize, see the paper [1] for details. 

