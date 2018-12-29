1.VIPLE简介   
  现在有许多优秀的用于计算和工程领域的可视化编程环境。ASU（美国亚利桑那州立大学）的 VIPLE (可视化程序设计语言) 是一个面向服务的软件开发环境，用于设计物联网以及基于多种硬件平台的机器人创建简易服务。其想法是让开发者绘制目标应用程序的流程图，而开发过程就是通过拖拽代表各个组件和服务的模块并将它们连接起来。这个简单的编程过程能让用户在几分钟创建自己的机器人应用程序  

2.安装VIPLE  
  VIPLE 是免费的，可以从 http://neptune.fulton.ad.asu.edu/VIPLE 下载。  
安装步骤：  
  第一步：在下载网站点击 ASU VIPLE Standard Edition Installer  
   ![image](https://github.com/QustRobot/RunCarInMazes/blob/master/images/8.PNG)   
  第二步：在弹出的页面点击 Install    
   ![image](https://github.com/QustRobot/RunCarInMazes/blob/master/images/9.PNG)   
  第三步：按照提示完成安装步骤，最后弹出 VIPLE 界面。   
 ![image](https://github.com/QustRobot/RunCarInMazes/blob/master/images/10.jpg)   
3.常用工具  
  基本活动：活动用于创建新的组件、服务、函数或其他代码模块。只需要简单的将一个活动拖至图中，打开它就可以组成一个新的组件。  
  服务：除了基本活动之外，VIPLE 也提供了很多内建的服务用以传统的输入和输出，也包括机器人相关的服务，比如传感器服、发动机和驱动服务。  
   ![image](https://github.com/QustRobot/RunCarInMazes/blob/master/images/11.jpg)   
4.简单编程示例  
  编程示例：使用计算机的键盘控制小车前后左右移动和停止。  
第一步：连接智能小车  
  搜索无线网络 QUST-ROBOT-xxx，输入密码：12345678 进行连接。  
   ![image](https://github.com/QustRobot/RunCarInMazes/blob/master/images/12.jpg)   
第二步：创建机器人主机服务  
  在服务中拖拽机器人主机到右边的工作区当中，之后会弹出 Connection type对话框，选择 Wi-Fi,点击确定按钮。   
   ![image](https://github.com/QustRobot/RunCarInMazes/blob/master/images/13.jpg)   
第三步：设置机器人主机的 TCP 端口号和 ip 地址   
  右键点击刚刚创建好的 My Robot 0，选择 Change TCP Port,在弹出的对话框中填入端口号：8124  
  再次右键点击 My Robot 0，选择 Properties，在弹出的对话框中填入 ip 地址：192.168.12.1  
   ![image](https://github.com/QustRobot/RunCarInMazes/blob/master/images/14.jpg) 
第四步：创建按键事件
  在服务中拖拽按键事件至右边的工作区中，在按键事件的下拉列表中选择 up，代表点击计算机方向键中的上方向键，同理选择 down 代表下方向键，left 表示左方向键，
right表示右方向键。

第五步：添加数据模块
  在基本活动中的数据拖拽到右边的工作区当中，并将按键事件与数据连接。数据 0.5表示以 0.5*小车最大速度向前移动，同理-0.5 表示以 0.5*小车最大速度向后移动，
所以数据范围是 [-1,1]。
  
第六步：为左轮和右轮设置速度
  在基本活动中拖拽与并，在与并中点击左下角的＋，分别在两个空白处填入变量名称 left 和 right（可随便定义变量名称），从数据引出两条线分别指向两个变量，表
示名称为left 变量赋值 0.5 的同时，名称为 right 变量也赋值为 0.5

第七步：添加机器人驱动器
  在服务中拖拽机器人服务器到右边的工作区，连接与并和机器人驱动器，之后弹出Data Connections 对话框，分别填入上一步定义的变量 left 和 right，表示将变量 left 
和 right里面保存的值（0.5）赋值给驱动器的左轮和右轮。

第八步：设置左轮和右轮控制端口
  右键点击机器人驱动器，选择 Properties，弹出 Robot Drive Settings 对话框，Partner 选择My robot 0，Left Wheel 填入 0，Right Wheel 填入 1 

第九步：测试
  点击 F5 功能键即可执行程序。按计算机的上方向键可以控制小车向前移动，点击 stop 小车停止

