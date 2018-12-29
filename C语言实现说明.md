# 1 系统定义  
## 1.1设计实现的功能  
  1.在小车本体上，用C编程，实现小车自主走迷宫及遥控程序。  
  2.手机遥控：通过已有手机APP（SPP/JuiceSSH），提供小车运动控制界面，通过手机APP界面控制小车的运动。  
## 1.2可行性分析  
  1.要完成迷宫程序，首先需要完成小车的避障和转弯行进两项基本功能。本车采用红外超声波避障，我们可以在原有小车的基础上，在它车头两侧分别加上一个红外感控装置，这个红外感控可以通过调试将测距避障的距离设置在最佳的位置。因为本车有三个超声波，也就是在小车的车头和两侧，能很好的检测周围赛道的情况。为了能更好的分析在测试中出现的故障，我们可以写两个测试红外和超声的代码，在小车运行中可以实时的传回数据。   
  2.而安卓端的实现，首先需要APP以及可以和小车进行联系的媒介。这里我们可以采用现成的APP（SPP/JuiceSSH）,通过手机wifi连接上小车的wifi，手机蓝牙作为串口通讯的媒介，这样就可以简单方便的实现安卓端的控制。   
## 1.3需求分析    
  1.跑迷宫的时候不允许采用遥控、人工干预等方式，需要小车自主跑出迷宫。不能卡住，也不能碰壁。  
  2.移动端遥控尽量简单易上手，成功率高。  
  
# 2 系统总体设计  
## 2.1总体设计方案的确定  
  树莓派智能小车服务主要实现的功能是实现避障功能，并接受手机客户端传过来的命令，并对命令做出正确的响应。  
  智能小车服务端的功能分两个模块实现， 分别为与手机客户端的通讯模块和小车运动模块。 这两个模块都是比较容易实现的， 小车端的通讯模块采用蓝牙串口通信。 小车的运动模块主要是通过小车中已经存在的中间件调用小车的 4个直流减速电机来实现小车的前进、 后退、 左转、 右转、 停止等功能。 小车的自动避障功能通过调用小车的超声波传感器来实现。   
  关于无线通信模块，有两种实现方式，一种是通过树莓派建立WIFI热点，然后通过手机连接树莓派的WIFI，由客户端建立TCP连接，实现对小车的控制。另一种是通过蓝牙，用手机连接树莓派上的蓝牙模块，然后将指令通过蓝牙串口透传到树莓派。两种方式各有优缺点，WIFI的方式可扩展性比较好，但是实现起来相对复杂，而且手机连接树莓派的WIFI会无法上网，而采用蓝牙串口的方式能避免这个问题，所以决定采用蓝牙串口的方式来实现树莓派小车的远程控制。  

## 2.2 软硬件功能划分   
  树莓派智能小车服务端要实现的主要功能是接收手机客户端传过来的命令， 并对命令做出正确的响应。命令主要是控制小车的模式，其中模式包含避障模式和遥控模式。命令通过蓝牙串口的方式进行传输，当切换到避障模式时，相关的逻辑由服务端的软件实现。   

## 2.3 软件体系架构设计  
 ![image](https://github.com/QustRobot/RunCarInMazes/blob/master/images/1.png)     
  由distance、motor、uart、task四个模块构成，通过串口转蓝牙通信在手机端实现对小车的测试超声波、测试红外、走迷宫、遥控四个功能。  
  distance：实现超声波测距及红外避障，包括初始化函数及距离检测更新函数，实现硬件功能的函数  
  motor：实现对电机的初始化，控制电机转动、停止的函数，包括电机的PWM调速的设置，用run()函数检测小车前、左、右方，使左右两侧的轮子速度同步，不需要考虑底层。    
  uart：配置串口，通过串口转成蓝牙协议，再通过手机app进行通信，本质上是串口通信，包括串口初始化和一些数据包的解析。  
  task：实现超声波检测距离，红外检测障碍，走迷宫，遥控四个功能，并整合到一个任务函数里。   
  main：调用前面定义的函数，初始化超声波测距，初始化红外避障，初始化电机，打开串口；再调用任务函数并接收发过去的数据，根据收到的命令标记任务，判断调用的功能函数。    

# 3 系统详细设计  
## 3.1 硬件详细设计
   系统的核心主控为树莓派3b,负责接收红外光电模块的数据，并控制电机驱动模块完成对四个直流减速电机的控制，此外它还能接收超声波模块的数据，来判断当前机器人的前方和两侧是否遇到障碍物。并且还能通过串口接收从手机客户端发过来的数据，实现无线遥控。  
   ![image](https://github.com/QustRobot/RunCarInMazes/blob/master/images/2.png)     
   电源模块采用LM2596S 开关电源稳压电路(支持 6~12 V 宽电压输入， 5 V 输出）  
   电机驱动模块： 
   ![image](https://github.com/QustRobot/RunCarInMazes/blob/master/images/3.png)     
   为了能实现最基本的避障功能，我们为小车配备了一些传感器。我们使用HC-SR04超声波测距传感器和红外光电避障传感器，其中红外光电传感器具有一堆红外线发射与接收管，运行时发射管会发出一定频率的红外线，当检测方向遇到障碍物时，红外线会反射回来被接收管接收，经过比较电路处理后，信号输出接口会输出一个低电平信号，只要在程序中对该接口的电平进行判断就能知道小车是否距离障碍物比较近。
无线模块：  
![image](https://github.com/QustRobot/RunCarInMazes/blob/master/images/4.png)     
采用串口模块直接接到树莓派的串口上，用来接收手机客户端发过来的数据，并对其中的命令做出响应，实现小车的无线遥控。   
## 3.2 软件详细设计  
解释主要代码：    
1、disMeasure()：距离检测。定义超声波引脚：Num 0: 前，Num 1: 左，Num 2: 右；发出超声波脉冲，根据时间差计算距离。  
```
//距离检测    
float disMeasure(unsigned int num)
{
struct timeval tv1;
struct timeval tv2;
long start, stop;
float dis;

if(num < 0 || num > 3)
return 0;

digitalWrite(Trig[num], LOW);
delayMicroseconds(2);

digitalWrite(Trig[num], HIGH);
delayMicroseconds(10); //发出超声波脉冲
digitalWrite(Trig[num], LOW);

while(!(digitalRead(Echo[num]) == 1));
gettimeofday(&tv1, NULL); //获取当前时间

while(!(digitalRead(Echo[num]) == 0));
gettimeofday(&tv2, NULL); //获取当前时间

start = tv1.tv_sec * 1000000 + tv1.tv_usec; //微秒级的时间
stop = tv2.tv_sec * 1000000 + tv2.tv_usec;

dis = (float)(stop - start) / 1000000 * 34000 / 2; //求出距离

return dis;
}
```
2、run()：电机转动，控制左右两侧的轮子同步。先限制一下速度范围，检测左侧两个电机的差值，检测右边两个电机的差值，控制轮子前进、停止、倒退。  
```
//电机转动
void run(int value_l, int value_r)
{
int v1, v2, v3, v4;

if(value_l>MAX_PWM) value_l = MAX_PWM;
if(value_r>MAX_PWM) value_r = MAX_PWM;
if(value_l<-1*MAX_PWM) value_l = -1*MAX_PWM;
if(value_r<-1*MAX_PWM) value_r = -1*MAX_PWM;

v1 = value_l>0 ? value_l : 0;
v2 = value_l<0 ? -value_l : 0;
v3 = value_r>0 ? value_r : 0;
v4 = value_r<0 ? -value_r : 0;

softPwmWrite(M_IN0,v1);
softPwmWrite(M_IN1,v2);
softPwmWrite(M_IN2,v3);
softPwmWrite(M_IN3,v4);

delay(10);
}

```

3、Check_Command()：接收串口通信发过去的数据，根据收到的命令用taskflag标志进行标记：  
0x01：检测超声波测距，taskflag标记为1；  
0x02：检测红外避障，taskflag标记为2；  
0x03：遥控功能，taskflag标记为3；  
0x04：迷宫功能，taskflag标记为4；  
0x00：关闭当前功能，taskflag标记回初始值0;  
如果是遥控功能还需要继续接收数据判断。  
```
//检查命令
void Check_Command(void)
{
if (serialDataAvail(fd) > 0)
{
UartBuff[0]=serialGetchar(fd);
switch(UartBuff[0])
{
case 0x01:
if(!taskflag)
{
taskflag = 1;
serialPuts(fd,"\r\nDistance Test\r\n");
}
break;
case 0x02:
if(!taskflag)
{
taskflag = 2;
serialPuts(fd,"\r\nInfrared Test\r\n");
}
break;
case 0x03:
if(!taskflag)
{
taskflag = 3;
serialPuts(fd,"\r\nRemote Mode\r\n");
}
break;
case 0x04:
if(!taskflag)
{
taskflag = 4;
serialPuts(fd,"\r\nAvoid Mode\r\n");
}
break;
case 0x00:
taskflag = 0;
serialPuts(fd,"\r\nClose Task\r\n");
stop(100);
break;
default:
if(taskflag == 3)
{
if(UartBuff[0]>0xF0) motormode = UartBuff[0]&0x0F;
}
else
{
taskflag = 0;
serialPuts(fd,"\r\nClose Task\r\n");
stop(100);
}

}
}

}
```


4、Car_Avoid()：走迷宫。  
   转弯分成两种情况：直角弯、修正弯。在触发“前超声波距离检测小于某值”这一条件时，判断左右距离差，若绝对值小于阈值，则未进入直角
弯，否则判断正负，若为正，则左边距离大，左转，即左轮翻反转，右轮正转，反之右转；在直行情况下，即“前超声波距离检测未超过某值”，而左右红外检测到障碍即
进入小弯调整，此时，右轮受左传感器影响，左轮受右传感器影响，哪边检测到障碍就停掉对边的轮子去修正直行。  
```
//走迷宫  
void Car_Avoid(void)
{
float temp;
int SL, SR;
//有信号为LOW 没有信号为HIGH
SR = digitalRead(RIGHT);//
SL = digitalRead(LEFT);//

Get_dis();
if(dis[Dis_Front] < 18.0)
{
Get_dis();
temp = dis[Dis_Left]-dis[Dis_Right];
printf("%0.2f\n",temp);
if(abs(temp) > 10.0)
{
stop(100);
if(temp > 0)
{
run(-350,350);
}
else
{
run(350,-350);
}
delay(350);
}
else
{
run(400,400);
}
}
else
{
printf("%d,%d\n",SR,SL);
if(SL == HIGH && SR == HIGH)
{
run(500,500);
}
else
{
run(400*SR, 400*SL);
}

}
}
```


5、Test_Motor()：遥控功能。接收串口通信发过去的数据，根据协议去控制小车：
F1：前进  
F2：后退  
F3：左转  
F4：右转  
F5：加速  
F6：减速  
F7：转向加速  
F8：转向减速  
FF：停止  
```
//遥控
void Test_Motor()
{
switch(motormode)
{
case 0x0f:stop(100); break;
case 0x01:run(speed,speed); break;
case 0x02:run(-1*speed,-1*speed); break;
case 0x03:run(-1*turn,turn); break;
case 0x04:run(turn,-1*turn); break;
case 0x05:if(speed<=MAX_PWM-10) speed+=10;motormode=0x00; break;
case 0x06:if(speed>10) speed-=10;motormode=0x00; break;
case 0x07:if(turn<=MAX_PWM-10) turn+=10;motormode=0x00; break;
case 0x08:if(turn>10) turn-=10;motormode=0x00; break;
case 0x09:stop(100); break;
case 0x00:serialPrintf(fd,"speed:%d,turn:%d\n",speed,turn);motormode=0x0f;break;
default:stop(100);
}
}
```

# 4 功能测试  
## 4.1.对c程序进行编译和链接，生成可执行文件  
```
$ gcc -c *.c
$ gcc *.o -o run -lwiringPi -lpthread
$ ./run
```

## 4.2.测试超声波测距：   

在“蓝牙串口”app点击“超声波”按键，其串口通信界面不断返回“front = ？ cm left = ？ cm right = ？ cm”字样（“？”为具体数值）
在前、左、右方的超声波进行不同距离的遮挡，看返回数据是否随遮挡距离变化。  
## 4.3.测试红外避障：  
在“蓝牙串口”app点击“红外”按键，其串口通信界面不断返回“Left:1,Right:1”字样  
对两个红外进行遮挡，若“1”变为“0”，则红外好用。  
## 4.4.测试遥控功能：  
在“蓝牙串口”app点击“遥控”按键，其串口通信界面不断返回“speed:？,turn:？”字样（“？”为具体数值，speed为直行速度，turn为转向速度）  
转到“键盘”窗口，按键控制小车。  
F1：前进  
F2：后退  
F3：左转  
F4：右转  
F5：加速  
F6：减速  
F7：转向加速  
F8：转向减速  
FF：停止  
## 4.5.测试走迷宫功能：  
在“蓝牙串口”app点击“迷宫”按键，小车即可自行走迷宫。  
## 4.6.手机端app控制界面与串口协议见图  
![image](https://github.com/QustRobot/RunCarInMazes/blob/master/images/5.png)
![image](https://github.com/QustRobot/RunCarInMazes/blob/master/images/6.png)  

## 4.7、附上迷宫图样  
![image](https://github.com/QustRobot/RunCarInMazes/blob/master/images/7.png)  
