# 1 系统定义   
## 1.1设计实现的功能    
python版：    
  只包含自动跑迷宫，无远程控制。   
c语言版：   
  自动跑迷宫+tcp客户端远程控制。   
## 1.2可行性分析     
  青科智能小车主要由一块树莓派核心控制板、四个直流减速电机、两节锂电池、三个超声波传感器、两个不怕光红外传感器和两个底部的光传感器组成。超声波传感和红外传感器可以实现避障功能，底部的光传感器可以实现循迹功能。核心控制板采用树莓派三代 B 型主板，速度更快、性能更强大，系统基于 Linux，已具备了电脑的有基本功能，通过编写相应程序便可实现所需的功能。   
## 1.3 需求分析  
  障碍物检测是智能小车导航研究中很重要的一个部分。在小车实际运行中，传感器相当于小车的“眼睛"，必须得到障碍物及其距离的信息，才能相应的规划自动避障导航算法。超声波测距是通过测量超声波从发射到遇到障物反射到被接收这整个过程中的时间差来确定距离，超声波传感器使用比较方便，具有信息处理简单，实时性强等特点，本设计便是基于超声波测距检测，编写相应的自动跑迷宫程序。  
# 2 系统总体设计  
## 2.1 总体设计方案的确定  
  基于超声波测距进行障碍物检测，用C语言编写TCP客户端远程控制程序，用python和C分别编写跑迷宫程序。  
## 2.2 软硬件功能划分   
### 2.2.1 硬件部分：   
  基于树莓派的智能小车一部，通过编写相应的程序使其实现自动跑迷宫等功能：  
  PC机一台，通过PC机接入小车的系统中，在PC机上直接对小车进行编程。  
### 2.2.2软件部分：   
python版：  
move.py：定义封装小车前进、后退、左右转等函数；  
ultrasonic.py：定义封装小车三个超声波测距函数；  
main.py：主进程,负责调用ultrasonic.py进行测距，从而调度move.py来控制小车移动；  
c语言版：  
car_server.c 为c语言程序：包含tcp服务器以及自动跑迷宫程序；  
tcp_client.py为python程序：是小车的tcp客户端；  

# 3 系统详细设计  
## 3.1 硬件详细设计     
核心控制板：青科智能车核心控制板采用树莓派三代 B 型主板，速度更快、性能更强大，只有信用卡大小的卡片式电脑，其系统基于 Linux。  
核心控制板参数配置：  
1.2 GHz 四核 Broadcom BCM2837 64 位 ARMv8 处理器  
板载 BCM43143 WiFi  
板载低功耗蓝牙 4.1 适配器(BLE)  
1 GB RAM  
4 个 USB 2 端口  
微型 SD 端口，用于存储操作系统及数据  
CSI 摄像头端口用于连接树莓派摄像头  
电机：  
L298N 电机驱动芯片(可直接驱动智能车底盘四个电机并可提供PWM 使能信号)  
4 个 1:48 抗干扰直流减速电机（工作电压 3-6 V）  
电源：  
LM2596S 开关电源稳压电路(支持 6~12 V 宽电压输入，5 V输出)  
1个船型电源控制开关(可对整个系统起到很好的电源管理作用)  
2 节 5000 mA 大容量 26650 电池，支持小车持续运转一个小时  
超声波传感器：  
电压：DC 5V；静态电流：小于 2 mA  
感应角度：不大于 15 度；探测距离：2 cm-450 cm   
高精度：可达 0.3 cm   
## 3.2 python版：  
```
from move import *
from ultrasonic import *
 
def main():
  ""转弯算法：左右前无障碍物：十字路口，左转、直行、右转，防止死循环
              左右无障碍物：丁字路口，左转、右转
              左边障碍物：右转、直行
              右边障碍物：左转、直行
              只有左边无障碍物：左转
              只有右边有障碍物：右转
              左右前均有障碍物：进入死角，后退到前一个路口，根据实际路口类型转向
              故：需封装前进，后退，左转，右转，后退以及超声波测距函数（反复调用）""
    s = t = z = y = 0
    while True:
        # 左右前无障碍物：十字路口，左转、直行、右转，防止死循环
        if dis(gpio_TRIGGER_FORWARD, gpio_ECHO_FORWARD) is False \
                and dis(gpio_TRIGGER_LEFT, gpio_ECHO_LEFT) is False \
                and dis(gpio_TRIGGER_RIGHT, gpio_ECHO_RIGHT) is False:
            s += 1
            if s % 3 == 1:
                left(10)
            elif s % 3 == 2:
                forward(2)
            elif s % 3 == 0:
                right(2)
 
        # 左右无障碍物：丁字路口，左转、右转
        elif dis(gpio_TRIGGER_FORWARD, gpio_ECHO_FORWARD) is True \
                and dis(gpio_TRIGGER_LEFT, gpio_ECHO_LEFT) is False \
                and dis(gpio_TRIGGER_RIGHT, gpio_ECHO_RIGHT) is False:
            t += 1
            if t % 2 == 1:
                left(2)
            elif t % 2 == 0:
                right(10)
 
        # 左边障碍物：右转、直行
        elif dis(gpio_TRIGGER_FORWARD, gpio_ECHO_FORWARD) is False \
                and dis(gpio_TRIGGER_LEFT, gpio_ECHO_LEFT) is True \
                and dis(gpio_TRIGGER_RIGHT, gpio_ECHO_RIGHT) is False:
            z += 1
            if z % 2 == 1:
                right(2)
            elif z % 2 == 0:
                forward(2)
 
        # 右边障碍物：左转、直行
        elif dis(gpio_TRIGGER_FORWARD, gpio_ECHO_FORWARD) is False \
                and dis(gpio_TRIGGER_LEFT, gpio_ECHO_LEFT) is False \
                and dis(gpio_TRIGGER_RIGHT, gpio_ECHO_RIGHT) is True:
            y += 1
            if y % 2 == 1:
                left(2)
            elif y % 2 == 0:
                forward(2)
 
        # 只有左边无障碍物：左转
        elif dis(gpio_TRIGGER_FORWARD, gpio_ECHO_FORWARD) is True \
                and dis(gpio_TRIGGER_LEFT, gpio_ECHO_LEFT) is False \
                and dis(gpio_TRIGGER_RIGHT, gpio_ECHO_RIGHT) is True:
            left(2)
 
        # 只有右边无障碍物：右转
        elif dis(gpio_TRIGGER_FORWARD, gpio_ECHO_FORWARD) is True \
                and dis(gpio_TRIGGER_LEFT, gpio_ECHO_LEFT) is True \
                and dis(gpio_TRIGGER_RIGHT, gpio_ECHO_RIGHT) is False:
            right(2)
 
        # 左右前均有障碍物：进入死角，后退到前一个路口，根据实际路口类型转向
        elif dis(gpio_TRIGGER_FORWARD, gpio_ECHO_FORWARD) is True \
                and dis(gpio_TRIGGER_LEFT, gpio_ECHO_LEFT) is True \
                and dis(gpio_TRIGGER_RIGHT, gpio_ECHO_RIGHT) is True:
            back(2)
  
if __name__ == "__main__":
    main()

控制小车移动部分：
import time
import RPi.GPIO as gpio
 
gpio.setwarnings(False)
 
def init():
    """端口初始化函数"""
    gpio.setmode(gpio.BCM)    # 使用 BOARD编码方式
    gpio.setup(12,gpio.OUT)   # 引脚 12 设置为输出
    gpio.setup(16,gpio.OUT)   # 引脚 16 设置为输出
    gpio.setup(18,gpio.OUT)   # 引脚 18 设置为输出
    gpio.setup(22,gpio.OUT)   # 引脚 22 设置为输出
  
def forward(run_time):
    """前进函数"""
    init()
    gpio.output(12, True)   # 对应IN1，False 代表不执行动作
    gpio.output(16, False)  # 对应IN2
    gpio.output(18, True)   # 对应 IN3，True代表右轮前进
    gpio.output(22, False)  # 对应IN4
    time.sleep(run_time)    # 引脚当前状态保持 runtime 秒
    gpio.cleanup()          # 清空引脚状态
 
def back(run_time):
    """后退函数"""
    init()
    gpio.output(12, False)  # 对应IN1，False 代表不执行动作
    gpio.output(16, True)   # 对应IN2
    gpio.output(18, False)  # 对应 IN3，True代表右轮前进
    gpio.output(22, True)   # 对应IN4
    time.sleep(run_time)    # 引脚当前状态保持 runtime 秒
    gpio.cleanup()          # 清空引脚状态
  
def left(run_time):
    """左转函数:IN1 左轮前进；IN2 左轮后退；
                IN3 右轮前进；IN4 右轮后退"""
    init()
    gpio.output(12, False)  # 对应IN1，False 代表不执行动作
    gpio.output(16, False)  # 对应IN2
    gpio.output(18, True)   # 对应 IN3，True代表右轮前进
    gpio.output(22, False)  # 对应IN4
    time.sleep(run_time)    # 引脚当前状态保持 runtime 秒
    gpio.cleanup()          # 清空引脚状态
  
def right(run_time):
    """右转函数"""
    init()
    gpio.output(12, True)   # 对应IN1，False 代表不执行动作
    gpio.output(16, False)  # 对应IN2
    gpio.output(18, False)  # 对应 IN3，True代表右轮前进
    gpio.output(22, False)  # 对应IN4
    time.sleep(run_time)    # 引脚当前状态保持 runtime 秒
    gpio.cleanup()          # 清空引脚状态
 
 
def stop(stop_time):
    """停止函数"""
    init()
    gpio.output(12, False)  # 对应IN1，False 代表不执行动作
    gpio.output(16, False)  # 对应IN2
    gpio.output(18, False)  # 对应 IN3，True代表右轮前进
    gpio.output(22, False)  # 对应IN4
    time.sleep(stop_time)   # 引脚当前状态保持 runtime 秒
gpio.cleanup()          # 清空引脚状态

超声波测距部分：
#!/usr/bin/python
# -*- coding: utf-8 -*-
import time
import RPi.GPIO as gpio
 
# Trig—超声波发送脚，高电平时发送出40KHZ出超声波
# ECHO—超声波接收检测脚，当接收到返回的超声波时，输出高电平
# 左边超声波：TRIG1 = 19；ECHO1 = 26
# 右边超声波：TRIG2 = 5；ECHO2 = 6
# 前面超声波：TRIG3 = 20；ECHO3 = 21
 
gpio.setmode(gpio.BCM)  # 使用BCM编码方式
 
# 前面超声波
gpio_TRIGGER_FORWARD = 20
gpio_ECHO_FORWARD = 21
 
# 左边超声波
gpio_TRIGGER_LEFT = 19
gpio_ECHO_LEFT = 26
 
# 前面超声波
gpio_TRIGGER_RIGHT = 5
gpio_ECHO_RIGHT = 6
 
# 设置引脚为输入和输出
gpio.setwarnings(False)
gpio.setup(gpio_TRIGGER_FORWARD, gpio.OUT)  # TRIGGER
gpio.setup(gpio_ECHO_FORWARD, gpio.IN)  # ECHO
 
gpio.setup(gpio_TRIGGER_LEFT, gpio.OUT)  # TRIGGER
gpio.setup(gpio_ECHO_LEFT, gpio.IN)  # ECHO
 
gpio.setup(gpio_TRIGGER_RIGHT, gpio.OUT)  # TRIGGER
gpio.setup(gpio_ECHO_RIGHT, gpio.IN)  # ECHO
 
 
def dis(gpio_TRIGGER,gpio_ECHO):  # 测距函数
    sign = False
    gpio.output(gpio_TRIGGER, False)  # 设置TRIGGER为低电平
    time.sleep(0.5)
    gpio.output(gpio_TRIGGER, True)  # 设置TRIGGER为高电平
    time.sleep(0.00001)
    gpio.output(gpio_TRIGGER, False)
    start = time.time()  # 记录发射超声波开始时间
 
    while gpio.input(gpio_ECHO) == 0:
        start = time.time()
 
    while gpio.input(gpio_ECHO) == 1:
        stop = time.time()  # 记录接收到超声波时间
 
    elapsed = stop - start  # 计算一共花费多长时间
    distance = elapsed * 34300  # 计算距离，就是时间乘以声速
    distance = distance / 2  # 除以2得到一次的距离而不是来回的距离
    print("Distance : %.1fcm" % distance)
    if distance < 10:
        sign = True
    else:
        sign = False
    return sign
 
# sign1 = dis(gpio_TRIGGER_FORWARD, gpio_ECHO_FORWARD)  # 调用测距函数
# time.sleep(0.5)
# sign2 = dis(gpio_TRIGGER_LEFT, gpio_ECHO_LEFT)        # 调用测距函数
# time.sleep(0.5)
# sign3 = dis(gpio_TRIGGER_RIGHT, gpio_ECHO_RIGHT)      # 调用测距函数
# time.sleep(0.5)
```


## 3.2 C语言版：
### TCP客户端：
```
import socket 
def main():
    
# 创建TCP套接    
tcp_socket = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
     
# 链接服务器
server_addr = ("192.168.12.1",3333)    
tcp_socket.connect(server_addr)
    
# 发送数据
while True:
send_data = input("请输入要发送的数据：")
tcp_socket.send(send_data.encode("utf-8"))
     
# 关闭套接字    
tcp_socket.close()
  
if __name__ == "__main__":
main()
```
### 服务端部分：
```
#include <stdlib.h>
#include <errno.h>
#include <string.h>

#include <unistd.h>
#include <netinet/in.h>
#include <fcntl.h>
#include <termios.h>
#include <stdio.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <sys/ioctl.h>
#include <pthread.h>
#include <sys/socket.h>
#include <wiringPi.h>
#include <softPwm.h>
#include <time.h>
#include <signal.h>

#define Trigf	28
#define Echof	29
#define Trigl	24
#define Echol	25
#define Trigr	21
#define Echor	22

#define BUFSIZE 512


#define SERVPORT 3333
#define BACKLOG 10
#define MAX_CONNECTED_NO 10
#define MAXDATASIZE 64

float disf,disl,disr;
int actflag = 0;
static int client_fd;
static char buf[MAXDATASIZE];
int flag=0;
int ret;
pthread_t id1,id2,id3;
 


void ultraInit(void)
{
    pinMode(Echof, INPUT);
    pinMode(Echor, INPUT);
    pinMode(Echol, INPUT);
    pinMode(Trigf, OUTPUT);
    pinMode(Trigr, OUTPUT);
    pinMode(Trigl, OUTPUT);
}


float disMeasure(int flr)
{
    struct timeval tv1;
    struct timeval tv2;
    long start, stop;
    float dis;

    if(flr==1)
    {
	digitalWrite(Trigf, LOW);
	delayMicroseconds(2);

	digitalWrite(Trigf, HIGH);
	delayMicroseconds(10);	  //发出超声波脉冲
	digitalWrite(Trigf, LOW);
  
	while(!(digitalRead(Echof) == 1));
	gettimeofday(&tv1, NULL);		   //获取当前时间

	while(!(digitalRead(Echof) == 0));
	gettimeofday(&tv2, NULL);		   //获取当前时间

	start = tv1.tv_sec * 1000000 + tv1.tv_usec;   //微秒级的时间
	stop  = tv2.tv_sec * 1000000 + tv2.tv_usec;

	dis = (float)(stop - start) / 1000000 * 34000 / 2;  //求出距离
    }

    else if(flr==2)
    {
	digitalWrite(Trigl, LOW);
	delayMicroseconds(2);

	digitalWrite(Trigl, HIGH);
	delayMicroseconds(10);	  //发出超声波脉冲
	digitalWrite(Trigl, LOW);
  
	while(!(digitalRead(Echol) == 1));
	gettimeofday(&tv1, NULL);		   //获取当前时间

	while(!(digitalRead(Echol) == 0));
	gettimeofday(&tv2, NULL);		   //获取当前时间

	start = tv1.tv_sec * 1000000 + tv1.tv_usec;   //微秒级的时间
	stop  = tv2.tv_sec * 1000000 + tv2.tv_usec;

	dis = (float)(stop - start) / 1000000 * 34000 / 2;  //求出距离

     }

     else if(flr==3)
    {
	digitalWrite(Trigr, LOW);
	delayMicroseconds(2);

	digitalWrite(Trigr, HIGH);
	delayMicroseconds(10);	  //发出超声波脉冲
	digitalWrite(Trigr, LOW);
  
	while(!(digitalRead(Echor) == 1));
	gettimeofday(&tv1, NULL);		   //获取当前时间

	while(!(digitalRead(Echor) == 0));
	gettimeofday(&tv2, NULL);		   //获取当前时间

	start = tv1.tv_sec * 1000000 + tv1.tv_usec;   //微秒级的时间
	stop  = tv2.tv_sec * 1000000 + tv2.tv_usec;

	dis = (float)(stop - start) / 1000000 * 34000 / 2;  //求出距离
    }

    return dis;
}


void run()     // 前进
{
    softPwmWrite(4,0); //左轮前进
    softPwmWrite(1,250); 
    softPwmWrite(6,0); //右轮前进
    softPwmWrite(5,250);  
}


void brake(int time)         //刹车，停车
{
    softPwmWrite(1,0); //左轮stop
    softPwmWrite(4,0); 
    softPwmWrite(5,0); //stop
    softPwmWrite(6,0); 

    delay(time * 100);//执行时间，可以调整  
}


void left(int time)         //左转(左轮不动，右轮前进)
{
    softPwmWrite(1,0); //左轮stop
    softPwmWrite(4,250); 
    softPwmWrite(5,250); //右轮前进
    softPwmWrite(6,0); 

    delay(time * 410);
}


void right(int time)        //右转(右轮不动，左轮前进)
{
    softPwmWrite(1,250); //左轮前进
    softPwmWrite(4,0); 
    softPwmWrite(5,0); //右轮stop
    softPwmWrite(6,250); 

    delay(time * 410);	//执行时间，可以调整  
}


void back(int time)          //后退
{
    softPwmWrite(4,500); //左轮back
    softPwmWrite(1,0); 
    softPwmWrite(6,500); //右轮back
    softPwmWrite(5,0); 

    delay(time *100);     //执行时间，可以调整  
}



void actThread()
{
    printf("actThread start!!\n");
    while(1)
    {
        switch(actflag)
        {
        case 0:brake(1);break;
        case 1:run();break;
        case 2:back(1);break;
        case 3:left(1);break;
        case 4:right(1);break;
	default:break;
        }
    }
pthread_exit(0);
}

void disThread()
{
    printf("disThread start!!\n");
    while(1)
    {
        if(flag == 1)
        {
            disf = disMeasure(1);
            disl = disMeasure(2);
            disr = disMeasure(3);
            printf("front distance = %0.2f cm\n",disf);//输出当前超声波测得的距离
            printf("left distance = %0.2f cm\n",disl);//输出当前超声波测得的距离
            printf("right distance = %0.2f cm\n\n",disr);//输出当前超声波测得的距离
        }
       else
       {
           printf("disThread stopped!!\n");
           pthread_exit(0);
       }
    }
}

void automatThread()
{
    printf("automatThread start!!\n");
    while(1)
    {
        if(flag==1)
        {
          if(disf<20||disf>2000)//测得前方障碍的距离小于20cm时做出如下响应
          {   
	    if((disr<20||disr>2000)&&(disl<20||disl>2000))
	    {
		brake(1);
		back(5);
		left(2);
	    }
	    else if(disr<disl)
	    {
		brake(1);
		back(0);
		left(2);
	    }
	    else 
	    {
		brake(1);
		back(0);
		right(2);
	    }
    	  }
          else
          {
            run();  //无障碍时前进
          }
        }
        else
        {
            printf("automatThread stopped!!\n");
            pthread_exit(0);
        }
    }
}
void automat()
{
    	 ret=pthread_create(&id3,NULL,(void *) automatThread,NULL);   //auto xiancheng
				if(ret!=0)
				{
				   printf("Create pthread error\n");
				   exit(1);
				}
}
int main()
{
    wiringPiSetup();
    /*WiringPi GPIO*/
    pinMode (1, OUTPUT);	//IN1
    pinMode (4, OUTPUT);	//
    pinMode (5, OUTPUT);	//IN3
    pinMode (6, OUTPUT);	//IN4
    softPwmCreate(1,1,500);
    softPwmCreate(4,1,500);
    softPwmCreate(5,1,500);
    softPwmCreate(6,1,500);

	struct sockaddr_in server_sockaddr,client_sockaddr;
	int sin_size=1,recvbytes;
	int sockfd;//client_fd;

	 ret=pthread_create(&id1,NULL,(void *) actThread,NULL); // dongzuo xiancheng
				if(ret!=0)
				{
				   printf("Create pthread error\n");
				   exit(1);
				}
     ret=pthread_create(&id2,NULL,(void *) disThread,NULL);   //chaoshengbo xiancheng
				if(ret!=0)
				{
				   printf("Create pthread error\n");
				   exit(1);
				}
	if((sockfd=socket(AF_INET,SOCK_STREAM,0))==-1)
	{
		perror("socket");
		exit(1);
	}
	printf("socket success! sockfd=%d\n",sockfd);

	server_sockaddr.sin_family=AF_INET;
	server_sockaddr.sin_port=htons(SERVPORT);
	server_sockaddr.sin_addr.s_addr=INADDR_ANY;
	bzero(&(server_sockaddr.sin_zero),8);

	if(bind(sockfd,(struct sockaddr *)&server_sockaddr,sizeof(struct sockaddr))==-1)
	{
		perror("bind");
		exit(1);
	}
	printf("bind success!\n");
	if(listen(sockfd,BACKLOG)==-1)
	{
		perror("listen");
		exit(1);
	}
	printf("listen success\n");
	if((client_fd=accept(sockfd,(struct sockaddr *)&client_sockaddr,&sin_size))==-1)
	{
		perror("accept");
		exit(1);
	}
	printf("accept success\n");

	while(1)
	{
		if((recvbytes=recv(client_fd,buf,1,0))>0)  //jieshoushuju chuli
		{
			printf("Receive:%s\n",buf);
			switch(buf[0])
            {
            case '0':flag = 0;actflag = 0;break;
            case '1':actflag = 1;break;
            case '2':actflag = 2;break;
            case '3':actflag = 3;break;
            case '4':actflag = 4;break;
            case '5':flag = 1;automat();break;
            default:break;
            }
		}

		else if(recvbytes<=0)
		{
			printf("Connection closed!\n");
			close(client_fd);
			client_fd=accept(sockfd,(struct sockaddr *)&client_sockaddr,&sin_size);
		}
	}
	printf("all over\n");
	pthread_join(id1,NULL); //??
	pthread_join(id2,NULL);  //??
	close(sockfd);
}
```

