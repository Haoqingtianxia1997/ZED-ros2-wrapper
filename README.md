<h1 align="center">
   <img src="./images/Picto+STEREOLABS_Black.jpg" alt="Stereolabs" title="Stereolabs" /><br>
   ZED ROS 2 Wrapper 中文说明
</h1>

<p align="center">
  用于在 ROS 2 中使用 Stereolabs ZED 相机的软件包。<br>
  ROS 2 Foxy Fitzroy（Ubuntu 20.04）— ROS 2 Humble Hawksbill（Ubuntu 22.04）— ROS 2 Jazzy Jalisco（Ubuntu 24.04）
</p>

本说明完整翻译仓库原英文 README，并补充当前工作区的使用说明和指定设备的启动命令。版本要求及生命周期日期保留原文信息；与当前源码不一致的地方另作说明。

本软件包支持在 ROS 2 中使用 ZED 相机，提供以下数据及功能：

- 校正和未校正的彩色、灰度图像。
- 深度数据。
- 彩色三维点云。
- 定位和建图，可选 GNSS 数据融合。
- 传感器数据。
- 目标检测结果。
- 人体骨架数据。
- 更多功能。

[更多信息](https://www.stereolabs.com/docs/ros2)

![深度点云](./images/PointCloud_Depth_ROS.jpg)

## 安装

### 前置条件

- [Ubuntu 20.04（Focal Fossa）](https://releases.ubuntu.com/focal/)、[Ubuntu 22.04（Jammy Jellyfish）](https://releases.ubuntu.com/jammy/) 或 [Ubuntu 24.04（Noble Numbat）](https://releases.ubuntu.com/noble/)。
- [ZED SDK](https://www.stereolabs.com/developers/release/latest/) v5.0 EA；旧版本支持请参见[历史发布版本](https://github.com/stereolabs/zed-ros2-wrapper/releases)。
- [CUDA](https://developer.nvidia.com/cuda-downloads) 依赖。
- ROS 2 Foxy Fitzroy（已弃用）、Humble Hawksbill 或 Jazzy Jalisco：
  - [Ubuntu 20.04 上安装 Foxy](https://docs.ros.org/en/foxy/Installation/Linux-Install-Debians.html)：**不推荐，已停止维护**。
  - [Ubuntu 22.04 上安装 Humble](https://docs.ros.org/en/humble/Installation/Linux-Install-Debians.html)：原文标注维护至 2027 年 5 月。
  - [Ubuntu 24.04 上安装 Jazzy](https://docs.ros.org/en/jazzy/Installation/Linux-Install-Debians.html)：原文标注维护至 2029 年 5 月。

### 编译软件包

**zed_ros2_wrapper** 使用 [colcon](http://design.ros2.org/articles/build_tool.html) 构建。

> 如果还没有建立 colcon 工作区，请先参考[工作区教程](https://index.ros.org/doc/ros2/Tutorials/Colcon-Tutorial/)。

打开 Bash 终端，加载已安装的 ROS 2 环境，然后克隆并编译仓库。以下保留原文的 `~/ros2_ws` 示例；本文后面的本地设备命令使用 `~/zed_ws`，请按实际工作区选择路径。已有本仓库时无需再次克隆。

```bash
source /opt/ros/humble/setup.bash # 示例：ROS 2 Humble
mkdir -p ~/ros2_ws/src/ # 工作区不存在时创建
cd ~/ros2_ws/src/ # 也可使用已有工作区的 src 目录
git clone https://github.com/stereolabs/zed-ros2-wrapper.git
cd ..
sudo apt update
rosdep update
rosdep install --from-paths src --ignore-src -r -y # 安装依赖
colcon build --symlink-install --cmake-args=-DCMAKE_BUILD_TYPE=Release --parallel-workers $(nproc)
echo source $(pwd)/install/local_setup.bash >> ~/.bashrc # 可选：新终端自动加载工作区
source ~/.bashrc
```

> `zed_msgs` 不再作为子模块安装。ROS 2 Humble 可通过 apt 安装对应二进制包；Foxy 或其他发行版可参考 [zed-ros2-interfaces 仓库](https://github.com/stereolabs/zed-ros2-interfaces?tab=readme-ov-file#install-the-package-from-the-source-code)从源码安装。

如果缺少 rosdep，可先安装：

```bash
sudo apt-get install python3-rosdep python3-rosinstall-generator python3-vcstool python3-rosinstall build-essential
```

在使用 JP6 的 NVIDIA Jetson 上首次编译时，可能遇到以下错误：

```text
CMake Error at /usr/share/cmake-3.22/Modules/FindCUDA.cmake:859 (message):
  Specify CUDA_TOOLKIT_ROOT_DIR
Call Stack (most recent call first):
 /usr/local/zed/zed-config.cmake:72 (find_package)
 CMakeLists.txt:81 (find_package)
```

可通过安装缺失的 JetPack 软件包解决：

```bash
sudo apt install nvidia-jetpack nvidia-jetpack-dev
```

`--symlink-install` 很重要：ROS 2 软件包需要安装节点使用的文件，该选项会在可行时以符号链接代替复制。这样修改工作区中无需编译的文件（如 Python 脚本、配置）后，下次运行即可使用更新内容，无需再次执行 `colcon build`。C++ 等需要编译的文件仍需重新构建。

如果使用 zsh，应将自动加载命令改为：

```zsh
echo source $(pwd)/install/local_setup.zsh >> ~/.zshrc
source ~/.zshrc
```

## 启动 ZED 节点

建议阅读 [ROS 2 DDS 与网络调优指南](https://www.stereolabs.com/docs/ros2/150_dds_and_network_tuning)，改善 ZED 在 ROS 2 中的使用体验。

打开 Bash 终端，通过[命令行工具](https://index.ros.org/doc/ros2/Tutorials/Introspection-with-command-line-tools/)启动：

```bash
ros2 launch zed_wrapper zed_camera.launch.py camera_model:=<camera_model>
```

将 `<camera_model>` 替换为实际型号：`zed`、`zedm`、`zed2`、`zed2i`、`zedx`、`zedxm`、`virtual`、`zedxonegs` 或 `zedxone4k`。

`zed_camera.launch.py` 是 Python 启动脚本，通过[手动组件组合](https://index.ros.org/doc/ros2/Tutorials/Composition/)启动 ZED 节点，并加载对应型号的 YAML 参数。同时启动 Robot State Publisher，根据相机型号对应的 URDF 模型发布静态连杆和关节关系。

可修改 `zed_wrapper/config` 下的配置文件：`common_stereo.yaml`、`zed.yaml`、`zedm.yaml`、`zed2.yaml`、`zed2i.yaml`、`zedx.yaml`、`zedxm.yaml`、`common_mono.yaml`、`zedxonegs.yaml` 和 `zedxone4k.yaml`。

使用 `-s` 查看所有启动参数：

```bash
ros2 launch zed_wrapper zed_camera.launch.py -s
ros2 launch zed_display_rviz2 display_zed_cam.launch.py -s
```

第二条命令需要另外安装示例仓库中的 `zed_display_rviz2`。各参数的完整说明见[配置参数指南](https://www.stereolabs.com/docs/ros2/zed_node#configuration-parameters)。

### RViz 可视化

[`zed-ros2-examples` 仓库中的 zed_display_rviz2](https://github.com/stereolabs/zed-ros2-examples/tree/master/zed_display_rviz2) 提供预配置 RViz 环境，可显示相机数据，并包含深度、点云、里程计、目标检测等高级可视化示例。

原文还提供以下命令，用于快速检查深度数据：

```bash
rviz2 -d ./zed_wrapper/config/rviz2/<your_camera_model>.rviz
```

**当前仓库没有 `zed_wrapper/config/rviz2` 目录**，因此只有在自行提供对应配置后才能使用该命令；也可安装上述示例包或手动配置 RViz。RViz 会订阅多个 ROS 话题，运行时可能影响应用性能。

### 仿真模式

> 该功能不支持 ZED X One 和第一代 ZED 相机。

使用模拟 ZED 数据作为输入，启动独立 ROS 2 节点：

```bash
ros2 launch zed_wrapper zed_camera.launch.py camera_model:=zedx sim_mode:=true
```

启动选项：

- **必填 `camera_model`**：模拟相机型号，必须与模拟器中的相机匹配。原文说明早期仿真插件主要模拟 ZED X，所以通常使用 `zedx`。
- **必填 `sim_mode`**：设置为 `true` 开启仿真模式。
- **可选 `use_sim_time`**：等待 `/clock` 上的有效消息，使用仿真时钟作为时间参考。
- **可选 `sim_address`**：仿真服务器地址，默认 `127.0.0.1`，适用于节点与模拟器运行在同一台机器上的情况。
- **可选 `sim_port`**：仿真服务器端口，必须与 `ZED camera streamer` Action Graph 节点属性中的 `Streaming Port` 一致。多相机仿真中，每台相机需使用不同端口。

还可以启动预配置 RViz，查看仿真提供的数据：

```bash
ros2 launch zed_display_rviz2 display_zed_cam.launch.py camera_model:=zedx sim_mode:=true
```

`display_zed_cam.launch.py` 包含 `zed_camera.launch.py`，因此可接收相同的启动参数。

下面展示将相机放置在模拟仓库货架上时，RViz 接收到的仿真数据：

![仿真 RViz](./images/sim_rviz.jpg)

![模拟仓库货架](./images/zed_shelves.jpg)

支持的仿真环境：

- [NVIDIA Omniverse Isaac Sim](https://www.stereolabs.com/docs/isaac-sim/)

## 更多功能

### SVO 录制

ZED 节点运行期间，可通过 `start_svo_recording` 和 `stop_svo_recording` 服务启动、停止 [SVO 录制](https://www.stereolabs.com/docs/video/recording/)。参见[服务说明](https://www.stereolabs.com/docs/ros2/zed_node/#services)。

### 目标检测

> 该功能不支持 ZED X One 和第一代 ZED 相机。

在 `common_stereo.yaml` 中将 `object_detection.od_enabled` 设置为 `true`，可在节点启动时自动开启目标检测。也可调用 `enable_obj_det` 服务手动开启或关闭。

详细说明见[目标检测文档](https://www.stereolabs.com/docs/ros2/object-detection)。原文使用 `object_detection/od_enabled` 表示配置层级，ROS 2 参数名称使用点号。

### 使用 YOLO 类 ONNX 模型进行自定义目标检测

> 该功能不支持 ZED X One 和第一代 ZED 相机。

目标检测支持使用 YOLO 类 ONNX 格式的自定义推理引擎。可以通过 Ultralytics YOLO 工具生成 ONNX 模型。

安装工具：

```bash
python -m pip install ultralytics
```

如果已经安装，可按原文方式更新：

```bash
pip install -U ultralytics
```

例如，将 YOLO 模型导出为 ONNX（参见[导出说明](https://docs.ultralytics.com/modes/export/)）：

```bash
yolo export model=yolo11n.pt format=onnx simplify=True dynamic=False imgsz=640
```

对于自行训练的模型，替换权重文件，例如：

```bash
yolo export model=yolov8l_custom_model.pt format=onnx simplify=True dynamic=False imgsz=512
```

更多信息参见 [Ultralytics 文档](https://github.com/ultralytics/ultralytics)。

修改 `common_stereo.yaml`，将 `object_detection.model` 设置为 `CUSTOM_YOLOLIKE_BOX_OBJECTS`，并根据实际模型调整 `custom_object_detection.yaml`。

> 首次使用自定义模型时，ZED SDK 会针对本机 GPU 优化模型，请等待优化结束。使用 Docker 时，建议通过共享卷将优化文件保存在宿主机，避免重复优化。

优化期间的控制台日志示例（英文提示表示正在针对显卡优化模型，通常只执行一次，可能需要数分钟）：

```text
[zed_wrapper-3] [INFO] [1729184874.634985183] [zed.zed_node]: === Starting Object Detection ===
[zed_wrapper-3] [2024-10-17 17:07:55 UTC][ZED][INFO] Please wait while the AI model is being optimized for your graphics card
[zed_wrapper-3]  This operation will be run only once and may take a few minutes
```

完整说明见[自定义目标检测文档](https://www.stereolabs.com/docs/ros2/custom-object-detection)。

### 人体跟踪

> 该功能不支持 ZED X One 和第一代 ZED 相机。

在 `common_stereo.yaml` 中将 `body_tracking.bt_enabled` 设置为 `true`，可在节点启动时自动开启人体跟踪（原文写作 `body_tracking/bt_enabled`）。

### 空间建图

> 该功能不支持 ZED X One。

在 `common_stereo.yaml` 中将 `mapping.mapping_enabled` 设置为 `true`，可在节点启动时自动开启空间建图（原文写作 `mapping/mapping_enabled`）。也可调用 `enable_mapping` 服务手动开启或关闭。

### GNSS 融合

> 该功能不支持 ZED X One。

Wrapper 可订阅 `NavSatFix` 话题，将 GNSS 数据与位置跟踪数据融合，获得相对于地球坐标的机器人定位。

原文说明通过 `gnss_fusion.gnss_fusion_enabled:=true` 启用，并要求启动时设置正确的 `gnss_frame`，例如 `gnss_frame:='gnss_link'`。**当前启动文件没有声明 `gnss_frame` 启动参数**，实际应以 `--show-args` 和当前配置为准。当前启动文件使用 `enable_gnss:=true` 覆盖融合开关，并提供 `gnss_antenna_offset` 设置天线相对相机安装点的位置。

`toLL` 和 `fromLL` 服务可用于经纬度坐标与机器人 `map` 坐标之间的转换。

### 二维模式

> 该功能不支持 ZED X One。

对于在平面上运动的机器人，可以通过 `common_stereo.yaml` 中的 `pos_tracking.two_d_mode` 开启二维模式（原文写作 `pos_tracking/two_d_mode`）。

里程计和位姿的 Z 坐标将固定为 `pos_tracking.fixed_z_value`（原文写作 `pos_tracking/fixed_z_value`）。横滚角 Roll、俯仰角 Pitch 及对应速度固定为零。

## NVIDIA® Isaac ROS 集成

Wrapper 兼容 [NVIDIA® Isaac ROS](https://nvidia-isaac-ros.github.io/) 框架，该框架提供在 NVIDIA 平台上构建机器人应用的工具与库。

Wrapper 使用 [NITROS](https://nvidia-isaac-ros.github.io/concepts/nitros/index.html)（NVIDIA Isaac Transport for ROS），在 NVIDIA 加速的 ROS 计算图中传输数据。

![NITROS 通信](./images/nitros-graph.gif)

参阅完整的 [Isaac ROS 集成指南](https://docs.stereolabs.com/isaac-ros/)，了解如何配置开发环境并结合 Isaac ROS、NITROS 使用 Wrapper。

## 示例与教程

示例和教程帮助理解 Wrapper 的使用方式及其与 ROS 2 的集成，参见 [zed-ros2-examples 仓库](https://github.com/stereolabs/zed-ros2-examples)。

### RViz2 可视化示例

- [zed_display_rviz2](https://github.com/stereolabs/zed-ros2-examples/tree/master/zed_display_rviz2)：通过启动文件打开预配置 RViz，显示 Wrapper 节点的数据。
- [rviz-plugin-zed-od](https://github.com/stereolabs/zed-ros2-examples/tree/master/rviz-plugin-zed-od)：用于 ZED2 的 ROS 2 插件，显示目标检测和人体跟踪结果，包括边界框与骨架。

### 教程

原文提供以下教程的相对链接。它们不在当前 Wrapper 仓库中，使用时请到上述示例仓库查找对应软件包。

- [视频订阅](./zed_video_tutorial)（`zed_video_tutorial`）：编写简单节点，订阅 `sensor_msgs/Image`，获取左右目的校正图像。
- [深度订阅](./zed_depth_tutorial)（`zed_depth_tutorial`）：订阅 `sensor_msgs/Image` 深度图，获取图像中心的距离测量值。
- [位姿与里程计订阅](./zed_pose_tutorial)（`zed_pose_tutorial`）：订阅 `geometry_msgs/PoseStamped` 和 `nav_msgs/Odometry`，获取相机运动时的位姿与里程计。
- [ROS 2 组件组合与 BGRA2BGR 转换](./zed_rgb_convert)（`zed_rgb_convert`）：使用组件组合和进程内通信，将四通道 BGRA 图像转换并发布为三通道 BGR 图像。
- [ROS 2 多相机](./zed_multi_camera)（`zed_multi_camera`）：使用提供的启动文件运行多相机机器人配置。
- [机器人集成](./zed_robot_integration)（`zed_robot_integration`）：向机器人配置添加一台或多台 ZED 相机。

### 应用示例

以下展示 ZED ROS 2 节点与其他软件包或高级功能结合使用的方式：

- [zed_aruco_localization](./zed_aruco_localization)：使用已定位的 ArUco 标记作为定位参考。
- [zed_depth_to_laserscan](./zed_depth_to_laserscan)：将 ZED 深度图转换为虚拟激光扫描。（原英文条目在 “using” 处截断，未提供后续细节。）

## 更新本地仓库

原文通过以下命令切换到 `master` 并拉取更新，并说明其用于获取 Wrapper 及子模块的最新提交：

```bash
git checkout master # 当前不在主分支时切换
git pull
```

如果实际检出版本使用了子模块，还需要按对应版本的子模块配置更新。当前工作副本有自定义配置，更新前请先提交或备份这些修改。

原文建议在重新编译前清理 colcon 工作区的构建缓存：

```bash
cd ~/ros2_ws # 替换为实际工作区根目录；本地设备配置使用 ~/zed_ws
rm -r install
rm -r build
rm -r log
colcon build --symlink-install --cmake-args=-DCMAKE_BUILD_TYPE=Release --parallel-workers $(nproc)
```

这里修正了原文将工作区根目录示例写成 `~/ros2_ws/src/` 的问题。清理命令会删除该工作区的安装产物、构建产物和日志，运行前应确认目录；以上仅为说明，本次文档修改未执行这些命令。

## 已知问题

原英文 README 的已知问题列表暂无报告。

发现缺陷或需要新功能时，可在 [GitHub Issues](https://github.com/stereolabs/zed-ros2-wrapper/issues) 提交问题。

---

## 当前工作区补充说明

以下保留本地使用指南，包含指定 ZED Mini 单相机、双相机的完整启动命令。

本仓库将 Stereolabs ZED 相机接入 ROS 2，可发布彩色图像、深度图、点云、IMU、位姿和里程计，并支持目标检测、人体跟踪等功能。可用功能取决于相机型号、SDK 和配置。

仓库包含三个 ROS 2 包：

| 包 | 用途 |
| --- | --- |
| `zed_wrapper` | 启动文件、相机参数和 URDF 模型 |
| `zed_components` | 相机驱动组件，负责采集和发布数据 |
| `zed_ros2` | 汇总依赖的元包 |

## 1. 环境准备与编译

以下命令以当前工作区 `~/zed_ws`、ROS 2 Humble 和 Bash 为例。原英文说明还列出了 Foxy、Jazzy 支持；使用其他发行版时，应替换环境加载路径，并核对依赖是否可用。

运行前需要：

- 已安装 ROS 2、colcon 和 rosdep。
- 已安装与硬件匹配的 NVIDIA 驱动、CUDA 和 ZED SDK。本仓库英文说明以 ZED SDK 5.0 EA 为前提，实际应确保 SDK 与当前源码兼容。
- 相机已连接，能够通过 ZED SDK 自带的工具正常打开。USB 相机应使用满足带宽要求的接口和线缆。

仓库已经位于 `~/zed_ws/src/zed-ros2-wrapper` 时，无需再次克隆：

```bash
source /opt/ros/humble/setup.bash
cd ~/zed_ws
rosdep update
rosdep install --from-paths src --ignore-src -r -y
colcon build --symlink-install --cmake-args=-DCMAKE_BUILD_TYPE=Release
source install/setup.bash
```

若 rosdep 尚未初始化，先运行一次 `sudo rosdep init`。如果提示找不到 `zed_msgs`，需安装对应 ROS 发行版的软件包，或将 `zed-ros2-interfaces` 源码放到工作区 `src` 后一起编译。

本地启动流程使用 Cyclone DDS；未安装时可执行：

```bash
sudo apt install ros-humble-rmw-cyclonedds-cpp
```

## 2. 当前设备的启动命令

### 加载工作区环境

每个用于启动相机或查看数据的新终端都先执行：

```bash
cd ~/zed_ws/
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
source src/zed-ros2-wrapper/setup_zed.sh
```

`setup_zed.sh` 会加载 ROS 2 Humble，设置 Cyclone DDS，并通过相对路径加载 `install/setup.bash`，因此执行时必须位于工作区根目录 `~/zed_ws`。如工作区不在主目录，请替换上述路径。

### 单相机（ZED Mini）

```bash
ros2 launch zed_wrapper zed_camera.launch.py camera_model:=zedm publish_tf:=false publish_map_tf:=false
```

默认相机名为 `zed`，节点通常为 `/zed/zed_node`。多台相机同时连接时，应使用下面的序列号参数明确选择设备。

### 双相机（两台 ZED Mini）

打开两个终端，分别执行前面的环境加载命令，再启动对应相机。设备对应关系如下：

| 位置 | 相机名 | 序列号 | 默认节点路径 |
| --- | --- | --- | --- |
| 右相机 | `right` | `14516188` | `/right/zed_node` |
| 左相机 | `left` | `11022812` | `/left/zed_node` |

右相机：

```bash
ros2 launch zed_wrapper zed_camera.launch.py camera_model:=zedm camera_name:=right serial_number:=14516188 publish_tf:=false publish_map_tf:=false
```

左相机：

```bash
ros2 launch zed_wrapper zed_camera.launch.py camera_model:=zedm camera_name:=left serial_number:=11022812 publish_tf:=false publish_map_tf:=false
```

这些命令关闭驱动发布的 `odom → camera_link` 和 `map → odom` 动态 TF。`publish_urdf` 默认仍为 `true`，所以相机内部的静态 TF 仍会发布。关闭 TF 发布不等于关闭定位计算；双相机与机器人之间的安装关系需要由机器人 URDF 或其他 TF 发布器提供。

可使用 SDK 工具 `ZED_Explorer -a` 查看已连接设备的序列号。更换设备后，请同步修改启动命令。

## 3. 确认数据是否正常

在另一个已加载相同环境的终端中执行：

```bash
ros2 node list
ros2 topic list
ros2 topic hz /zed/zed_node/rgb/image_rect_color
ros2 topic info /zed/zed_node/depth/depth_registered --verbose
```

常用双目相机话题如下。单相机默认前缀为 `/zed/zed_node`；上述双相机命令分别使用 `/right/zed_node` 和 `/left/zed_node`。

| 话题后缀 | 内容 |
| --- | --- |
| `/rgb/image_rect_color` | 校正后的彩色图像 |
| `/rgb/camera_info` | 相机内参 |
| `/left/image_rect_color` | 左目校正图像 |
| `/right/image_rect_color` | 右目校正图像 |
| `/depth/depth_registered` | 对齐后的深度图 |
| `/point_cloud/cloud_registered` | 彩色点云 |
| `/imu/data` | IMU 数据，取决于设备支持情况 |
| `/pose` | 位姿，需定位功能运行 |
| `/odom` | 里程计，需定位功能运行 |

例如查看右相机图像频率：

```bash
ros2 topic hz /right/zed_node/rgb/image_rect_color
```

深度图默认使用以米为单位的 32 位浮点数；`depth.openni_depth_mode:=true` 对应的配置会改为以毫米为单位的 16 位无符号整数。编写订阅程序时应检查消息的 `encoding`。

部分数据按订阅需求计算或发布；话题存在但暂时没有消息时，也应检查功能开关和启动日志。

## 4. 查看图像和点云

加载环境后启动 RViz：

```bash
rviz2
```

在 RViz 中添加 `Image` 查看图像，添加 `PointCloud2` 查看点云。将 `Fixed Frame` 设置为实际存在的 TF 坐标系，例如从点云消息的 `header.frame_id` 中获取；当前命令关闭了动态 TF，不能直接假设 `map` 可用。

如果无法显示，检查话题选择、TF 和 QoS；可尝试将显示项的 Reliability 设置为 `Best Effort`。

本仓库没有附带 `config/rviz2` 目录。需要官方预配置可视化启动文件时，请另外安装 `zed-ros2-examples` 中的 `zed_display_rviz2` 包。

## 5. 修改参数

配置文件位于 [zed_wrapper/config](zed_wrapper/config)：

| 文件 | 用途 |
| --- | --- |
| `common_stereo.yaml` | 双目相机通用配置：发布频率、深度、定位、建图等 |
| `common_mono.yaml` | 单目 ZED X One 的通用配置 |
| `zedm.yaml`、`zed2.yaml` 等 | 各型号的采集分辨率、采集帧率等 |
| `object_detection.yaml` | 目标检测配置 |
| `custom_object_detection.yaml` | 自定义检测模型配置 |

建议通过额外 YAML 覆盖需要调整的参数。例如创建 `~/zed_ws/zed_override.yaml`：

```yaml
/**:
  ros__parameters:
    general:
      grab_resolution: 'HD720'
      grab_frame_rate: 30
      pub_frame_rate: 30.0
    depth:
      point_cloud_freq: 10.0
```

在原启动命令后追加覆盖文件路径即可，例如：

```bash
ros2 launch zed_wrapper zed_camera.launch.py \
  camera_model:=zedm \
  publish_tf:=false publish_map_tf:=false \
  ros_params_override_path:=$HOME/zed_ws/zed_override.yaml
```

加载顺序为通用配置、型号配置、其他功能配置、覆盖文件，最后由启动参数覆盖其对应的值。例如 `publish_tf` 应通过 launch 参数设置。

采集分辨率与帧率必须是相机支持的组合，图像发布频率和点云频率不能高于采集频率。当前工作副本里有自定义帧率设置，不应直接将这些数值视为设备支持范围。

查看启动选项及节点实际参数：

```bash
ros2 launch zed_wrapper zed_camera.launch.py --show-args
ros2 param list /zed/zed_node
ros2 param get /zed/zed_node general.pub_frame_rate
ros2 param dump /zed/zed_node
```

标注 `[DYNAMIC]` 的参数通常支持运行时调整；其他参数修改后应重启节点。使用 `--symlink-install` 编译时，已有配置文件的内容修改通常可在下次启动时生效；C++ 源码变更仍需重新编译。

## 6. 其他启动方式与功能

`camera_model` 必须与设备一致，可选值为 `zed`、`zedm`、`zed2`、`zed2i`、`zedx`、`zedxm`、`virtual`、`zedxonegs`、`zedxone4k`。其中 `virtual` 需要额外设置正值 `custom_baseline`。

回放 SVO 文件（替换为实际文件路径和对应相机型号）：

```bash
ros2 launch zed_wrapper zed_camera.launch.py \
  camera_model:=zedm svo_path:=/absolute/path/recording.svo2
```

连接已经运行的相机仿真数据服务：

```bash
ros2 launch zed_wrapper zed_camera.launch.py \
  camera_model:=zedx sim_mode:=true sim_address:=127.0.0.1 sim_port:=30000
```

仿真命令不会启动模拟器；型号和端口需与模拟器一致。只有存在 `/clock` 发布者且需要使用该时钟时，才设置 `use_sim_time:=true`。

双目相机的可选功能可在覆盖文件中按需设置：

| 参数 | 作用 |
| --- | --- |
| `pos_tracking.pos_tracking_enabled` | 启动定位 |
| `pos_tracking.two_d_mode` | 平面运动模式 |
| `mapping.mapping_enabled` | 空间建图 |
| `object_detection.od_enabled` | 目标检测 |
| `body_tracking.bt_enabled` | 人体跟踪 |

功能支持受设备限制，例如第一代 ZED 不支持目标检测和人体跟踪。`depth.depth_mode` 为 `NONE` 会禁用依赖深度的功能；`depth.depth_stabilization` 大于零可能触发定位，即使 `pos_tracking_enabled` 为 `false`。

录制 SVO、重置定位等操作通过服务提供。先查询实际服务及接口，再按接口字段调用：

```bash
ros2 service list -t
# 将下方占位符替换为上一步查到的接口类型
ros2 interface show <服务接口类型>
```

## 7. 常见问题

- **找不到 `zed_wrapper`**：确认编译成功，并在当前终端加载了 `~/zed_ws/install/setup.bash`。
- **`setup_zed.sh` 找不到 `install/setup.bash`**：先进入 `~/zed_ws`，再 source 脚本。
- **找不到 Cyclone DDS**：安装 `ros-humble-rmw-cyclonedds-cpp`；相关终端应使用一致的中间件配置。
- **相机打不开**：检查设备连接、序列号、采集参数，以及是否被其他进程占用；先用 SDK 工具单独验证。
- **双相机只能启动一台或频率下降**：检查序列号是否不同、USB 控制器带宽和 GPU 负载，适当降低分辨率、采集频率、发布频率或点云频率。
- **有图像但没有点云或定位**：检查深度模式、定位开关、订阅情况及节点日志；TF 发布开关不会直接决定图像是否发布。
- **其他终端看不到话题**：确认已加载环境，`ROS_DOMAIN_ID`、DDS 配置及网络发现设置一致。
- **CPU/GPU 占用高**：减少不需要的订阅，降低图像或点云发布频率，并按需关闭检测、人体跟踪和建图。

使用 `Ctrl+C` 停止对应终端中的相机节点。

## 8. 参考资料

- [本地启动文件](zed_wrapper/launch/zed_camera.launch.py)
- [Docker 使用说明（英文）](docker/README.md)

本指南依据当前仓库的启动文件、源码和配置编写；更换版本后，请重新通过 `--show-args` 和节点参数确认实际选项。
