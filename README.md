# jubot_demo
# [从零开始仿真ROS小车（一）urdf模型+rviz可视化](https://blog.csdn.net/weixin_52029211/article/details/119538435)

## 一、创建功能包

```bash
cd catkin_ws #进入工作空间
mkdir src # 创建src文件夹
cd src    # 进入src，在此目录下创建功能包
catkin_create_pkg jubot_demo urdf xacro #创建功能包、添加依赖
cd jubot_demo/
mkdir urdf
mkdir launch
mkdir meshes #存放渲染机器人模型的文件
mkdir config #存放rviz配置的文件
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/e60e36ec52b44da0869d13d41ec4a0a5.png#pic_center)打开VS Code写两个文件  
`jubot_base.urdf`（放urdf文件夹下）  
`display_jubot_base_urdf.launch`（放launch文件夹下）  
config里的rviz文件是保存生成的，不用写

## 二、urdf文件

Unified Robot Description Format，统一机器人描述格式，简称为URDF  
模型的环节（link）与关节（joint）坐标关系，跟我的代码模型不匹配，仅供理解关系  
![在这里插入图片描述](https://img-blog.csdnimg.cn/5200a24a9ba94929a6615c5770aa9547.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81MjAyOTIxMQ==,size_16,color_FFFFFF,t_70#pic_center)在基础模型之上，我们为机器人添加尺寸大小。由于每个环节的参考系都位于该环节的底部，关节也是如此，所以在表示尺寸大小时，只需要描述其相对于连接的关节的相对位置关系即可。URDF中的 origin 域就是用来表示这种相对关系。

如果我们为机器人的关节添加 axis 旋转轴参数，那么该机器人模型就可以具备基本的运动学参数。

> 注意代码中不能有中文注释

```xml
<?xml version="1.0" ?>
<robot name="jubot">
    
<!--base_car-->>
    <link name="base_link">   
        <visual> 
            <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/> 
            <geometry>  
                <cylinder radius="0.20" length="0.16"/>             
            </geometry>
            <material name="yellow"> 
                <color rgba="1 0.4 0 1"/>  
            </material>
        </visual>
    </link>

<!--left_wheel-->>
    <joint name="left_wheel_joint" type="continuous">   
        <origin xyz="0.0 0.19 -0.05" rpy="0.0 0.0 0.0"/> 
        <parent link="base_link"/>  
        <child link="left_wheel_link"/> 
        <axis xyz="0.0 1.0 0.0"/>  
    </joint>

    <link name="left_wheel_link">
        <visual>
            <origin xyz="0.0 0.0 0.0" rpy="1.5707 0.0 0.0"/>
            <geometry>
                <cylinder radius="0.06" length="0.025"/>
            </geometry>
            <material name="white">
                <color rgba="1 1 1 0.9"/>
            </material>
        </visual>
    </link>
 
<!--right_wheel-->>
    <joint name="right_wheel_joint" type="continuous">
        <origin xyz="0.0 -0.19 -0.05"/>
        <parent link="base_link"/>
        <child link="right_wheel_link"/>
        <axis xyz="0.0 1.0 0.0"/>
    </joint>

    <link name="right_wheel_link">
        <visual>
            <origin xyz="0.0 0.0 0.0" rpy="1.5707 0.0 0.0"/>
            <geometry>
                <cylinder radius="0.06" length="0.025"/>
            </geometry>
            <material name="white">
                <color rgba="1 1.0 1.0 0.9"/>
            </material>
        </visual>
    </link>

<!--front_caster-->
    <joint name="front_caster_joint" type="continuous">
        <origin xyz="0.18 0.0 -0.095" rpy="0.0 0.0 0.0"/>
        <parent link="base_link"/>
        <child link="front_caster_link"/>
        <axis xyz="0.0 1.0 0.0"/>
    </joint>

    <link name="front_caster_link">
        <visual>
            <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
                <geometry>
                    <sphere radius="0.015"/>
                </geometry>
                <material name="black">
                    <color rgba="0.0 0.0 0.0 0.95"/>
                </material>
        </visual>
    </link>

<!--back_caster-->
    <joint name="back_caster_joint" type="continuous">
        <origin xyz="-0.18 0.0 -0.095" rpy="0.0 0.0 0.0"/>
        <parent link="base_link"/>
        <child link="back_caster_link"/>
        <axis xyz="0.0 1.0 0.0"/>
    </joint>

    <link name="back_caster_link">
        <visual>
            <origin xyz="0.0 0.0 0.0" rpy="0.0 0.0 0.0"/>
                <geometry>
                    <sphere radius="0.015"/>
                </geometry>
            <material name="black">
                <color rgba="0.0 0.0 0.0 0.95"/>
            </material>
        </visual>
    </link>
</robot>
```

关于参数欧拉角rpy，是roll（滚转角）、pitch（俯仰角）、yaw（偏航角），分别对应绕x轴、y轴、z轴  
![在这里插入图片描述](https://img-blog.csdnimg.cn/b0ac8da4adfc4b3d901998c7d17ee445.png#pic_center)

## 三、launch文件

```xml
<launch>
	<!-- 设置机器人模型路径参数 -->
	<param name="robot_description" textfile="$(find jubot_demo)/urdf/jubot_base.urdf" />

	<!-- 运行joint_state_publisher节点，发布机器人的关节状态  -->
	<node name="joint_state_publisher_gui" pkg="joint_state_publisher_gui" type="joint_state_publisher_gui" />
	
	<!-- 运行robot_state_publisher节点，发布tf  -->
	<node name="robot_state_publisher" pkg="robot_state_publisher" type="robot_state_publisher" />
	
	<!-- 运行rviz可视化界面 -->
    <node name="rviz" pkg="rviz" type="rviz" args="-d $(find jubot_demo)/config/jubot_urdf.rviz" required="true" />  
</launch>
```

## 四、图形化显示

安装一个检查urdf语法的工具：

```xml
sudo apt-get install liburdfdom-tools
```

在urdf文件夹下打开终端检查语法：

```xml
check_urdf jubot_base.urdf
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/f421e09826974adbadd3c3982456bcfb.png#pic_center)  
在urdf文件夹下打开终端，图形化显示URDF模型

```xml
urdf_to_graphiz jubot_base.urdf
```

此时会生成两个文件，打开pdf文件

![在这里插入图片描述](https://img-blog.csdnimg.cn/db4282e5c76145939becc9605700d5eb.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81MjAyOTIxMQ==,size_16,color_FFFFFF,t_70#pic_center)

## 五、RVIZ可视化

启动launch文件  
`roslaunch jubot_demo display_jubot_base_urdf.launch`  
启动rviz  
`rviz`  
![在这里插入图片描述](https://img-blog.csdnimg.cn/2c07b3db872c47d2833cfa2ff9955cb6.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81MjAyOTIxMQ==,size_16,color_FFFFFF,t_70#pic_center)  
以上圆形步骤就完成了，接下来我们再写一个方形小车

## 六、再写一个urdf模型

依然是一个urdf文件和launch文件  
跟上面的差不多，只是在参数上稍微做了修改，结构还是一个底座4个轮子  
![在这里插入图片描述](https://img-blog.csdnimg.cn/35271b5eda0049d8be73abca9e856a6c.png#pic_center)![](https://img-blog.csdnimg.cn/9987bddb52574b2c9b9fff587a8a74c5.png#pic_center)  
调节左下角参数轮子会转动

![在这里插入图片描述](https://img-blog.csdnimg.cn/a6c2066b04ca41d0a320789b8845d166.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81MjAyOTIxMQ==,size_16,color_FFFFFF,t_70#pic_center)第二版–加了柱子和第二层  
![在这里插入图片描述](https://img-blog.csdnimg.cn/c0819ec7936c453f91e013de251103f1.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl81MjAyOTIxMQ==,size_16,color_FFFFFF,t_70#pic_center)
