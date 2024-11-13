
### 在 Ubuntu 20.04 上搭建基于 VS Code 的 ROS Noetic 开发环境

本教程将帮助你在 Ubuntu 20.04 上配置 ROS Noetic 和 VS Code 开发环境，并进行简单测试。这个环境支持 **C++ 和 Python** 两种语言的开发、编译、运行和调试。

### 1. **安装 ROS Noetic**

#### 1.1 更新系统并安装必要工具

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install curl gnupg lsb-release -y
```

#### 1.2 配置 ROS 软件源

```bash
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -
```

#### 1.3 安装 ROS Noetic

```bash
sudo apt update
sudo apt install ros-noetic-desktop-full
```

#### 1.4 初始化 rosdep

```bash
sudo apt install python3-rosdep
sudo rosdep init
rosdep update
```

#### 1.5 设置环境变量

```bash
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

#### 1.6 安装 ROS 依赖工具

```bash
sudo apt install python3-rosinstall python3-rosinstall-generator python3-wstool build-essential
```

#### 1.7 测试 ROS 安装

启动 ROS 核心节点：

```bash
roscore
```

如果成功启动 `roscore`，说明安装正常。

### 2. **创建 ROS 工作空间**

#### 2.1 创建 `catkin_ws` 工作空间

```bash
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws
catkin_make
```

#### 2.2 配置环境变量

```bash
echo "source ~/catkin_ws/devel/setup.bash" >> ~/.bashrc
source ~/.bashrc
```

### 3. **安装 VS Code 和扩展**

#### 3.1 安装 VS Code

使用 Snap 安装：

```bash
sudo snap install --classic code
```

#### 3.2 安装 VS Code 插件

打开 VS Code，安装以下插件：

- **ROS** (by Microsoft): 提供 ROS 支持。
- **C/C++** (by Microsoft): 提供 C++ 语言支持。
- **Python** (by Microsoft): 提供 Python 开发支持。

### 4. **配置 VS Code 环境**

#### 4.1 配置 C++ 编译任务 (`tasks.json`)

在工作空间目录下创建 `.vscode/tasks.json` 文件：

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "catkin_make",
            "type": "shell",
            "command": "catkin_make",
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "problemMatcher": ["$gcc"],
            "detail": "Build the ROS workspace using catkin_make."
        }
    ]
}
```

#### 4.2 配置调试器 (`launch.json`)

在 `.vscode/launch.json` 文件中，添加以下内容：

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "Debug ROS Node",
            "type": "cppdbg",
            "request": "launch",
            "program": "${workspaceFolder}/devel/lib/<your_package>/<your_node>",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${workspaceFolder}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "catkin_make"
        }
    ]
}
```

- **program**: 修改为你需要调试的 ROS 节点可执行文件路径。

### 5. **创建示例 ROS 包并测试**

#### 5.1 创建一个 ROS 包

```bash
cd ~/catkin_ws/src
catkin_create_pkg my_robot std_msgs rospy roscpp
```

#### 5.2 创建 C++ 示例节点

新建 `my_robot/src/talker.cpp` 文件编写代码：
```cpp
#include "ros/ros.h"
#include "std_msgs/String.h"

int main(int argc, char **argv) {
    ros::init(argc, argv, "talker");
    ros::NodeHandle n;
    ros::Publisher chatter_pub = n.advertise<std_msgs::String>("chatter", 1000);
    ros::Rate loop_rate(1);

    while (ros::ok()) {
        std_msgs::String msg;
        msg.data = "Hello, ROS World!";
        ROS_INFO("%s", msg.data.c_str());
        chatter_pub.publish(msg);
        ros::spinOnce();
        loop_rate.sleep();
    }

    return 0;
}
```
在 `my_robot/CMakeLists.txt` 文件添加：
```sh
add_executable(talker src/talker.cpp)
target_link_libraries(talker ${catkin_LIBRARIES})
install(TARGETS talker
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
```

#### 5.3 编译示例节点

```bash
cd ~/catkin_ws
catkin_make
```

#### 5.4 运行节点并测试

启动 `roscore`：

```bash
roscore
```

在新的终端窗口中运行 `talker` 节点：

```bash
rosrun my_robot talker
```

你应该会看到类似以下的输出：

```
[INFO] [time]: Hello, ROS World!
```

#### 5.5 使用 `launch` 文件启动调试

如果你通过 `rosrun` 启动节点时无法调试，尝试创建一个 `launch` 文件，并在 `launch.json` 中使用 `launch` 文件启动节点。

在 `my_robot/launch/talker.launch` 文件中：

```xml
<launch>
    <node pkg="my_robot" type="talker" name="talker" output="screen" launch-prefix="gdb -ex run --args"/>
</launch>
```

然后使用以下命令启动：
```bash
roslaunch my_robot talker.launch
```

#### 5.6 使用 VS Code 调试节点

1. 打开 `talker.cpp` 文件，在 `ROS_INFO` 行上设置断点。
2. 点击调试图标，选择 **Debug ROS Node** 配置，点击 **Start Debugging** 或按 `F5`。
3. 程序会在断点处暂停，你可以使用 **Step Over (F10)**、**Step Into (F11)** 等功能进行调试。



### 6. **配置 Python ROS 节点支持**

如果你使用 Python 编写 ROS 节点，只需安装 Python 插件，并确保 Python 环境正确配置。

```sh
python --version
```
ROS Noetic 使用 Python 3.8，因此如果你的虚拟环境使用的是其他版本，建议重新创建虚拟环境并指定 Python 3.8：

#### 6.1 创建 Python 节点示例

在 `my_robot/scripts/listener.py` 中创建 Python 脚本：

```python
#!/usr/bin/env python3
import rospy
from std_msgs.msg import String

def callback(data):
    rospy.loginfo("I heard %s", data.data)

def listener():
    rospy.init_node('listener', anonymous=True)
    rospy.Subscriber("chatter", String, callback)
    rospy.spin()

if __name__ == '__main__':
    listener()
```

#### 6.2 赋予可执行权限并运行

```bash
chmod +x ~/catkin_ws/src/my_robot/scripts/listener.py
rosrun my_robot listener.py
```

在 `talker` 节点运行的情况下，`listener` 节点会输出：

```
[INFO] [time]: I heard Hello, ROS World!
```

---

### 总结

通过以上步骤，你现在已经搭建了一个完整的 ROS Noetic 开发环境，支持 C++ 和 Python 开发，并能够使用 VS Code 进行编译和调试。这个环境非常适合日常 ROS 开发和调试。如果有任何问题或者需要进一步配置，欢迎继续提问！