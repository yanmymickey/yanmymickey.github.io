---
title: Linux 实验 socket网络编程
date: 2020-11-05
tags: [Note,Linux,socket]
categories:
- [Note,Linux]
comments: false
---

# Linux 实验 socket网络编程

## 要求

编写两个程序，一个Server，一个Client，分别运行在两个shell窗口，通过Socket实现两个Shell窗口的即时通信

（即在A窗口输入Hello，B窗口会立即显示“Client：Hello”。B窗口也可以输入任意内容，比如“Nice to meet you”，在A窗口立即显示“Server： Nice to meet you”。）
（提示：本地地址是127.0.0.1，端口最好10000以上，避免占据其他应用端口。注意访问权限问题。）

<!-- more -->

## Server.c

```
#include <stdio.h>
#include <signal.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/wait.h>

#define PORT 23456
#define SERVER_ADDR "127.0.0.1"

int server_socket,client_socket;

void func_kill(int sig){
    // printf("%d ",close(server_socket));
    // printf("%d ",close(client_socket));
    close(server_socket);
    close(client_socket);
    signal(SIGINT,SIG_DFL);
    exit(3);
}

int main(int argc, char *argv[])
{
    // 俩个子进程 pid1 用于接收 pid2 用于发送
	int pid1,pid2;
    // 等待pid退出信号
    int pid_exit_status;
    // 绑定端口结果 监听端口结果
    int Bind_res,Listen_res;
    // 客户端地址块大小
	socklen_t clientaddr_len;
    // 服务端地址块 客户端地址块
	struct sockaddr_in serveraddr,clientaddr;
    // 存储接受的消息 发送的消息
	char recv_msg[100],send_msg[100];

	//创建服务socket
	server_socket = socket(AF_INET,SOCK_STREAM,0);
    if(server_socket==-1)
	{
		perror("The socket is error!\n");
		exit(0);
	}

    // 设置端口绑定,取消TIME_WAIT,避免占用
    struct linger ling;
    ling.l_onoff = 1;
    ling.l_linger = 0;
    setsockopt(server_socket,SOL_SOCKET, SO_LINGER, &ling, sizeof(ling));

	
    // 设置服务端地址
	serveraddr.sin_family = AF_INET; //ipv4
	serveraddr.sin_port = htons(PORT); //端口
    inet_aton(SERVER_ADDR,&serveraddr.sin_addr); //地址
    bzero(serveraddr.sin_zero,8);

	//绑定 和 监听 端口
	Bind_res  = bind(server_socket, (struct sockaddr *)(&serveraddr),sizeof(serveraddr));
	if(Bind_res==-1)
	{
		perror("The bind is error!\n");
		exit(0);
	}
	Listen_res = listen(server_socket,2);
	if(Listen_res==-1)
	{
		perror("The listen is error!\n");
		exit(0);
	}
	printf("server start!\n");
    
    // 捕获 ctrl+c信号
    signal(SIGINT,func_kill);

    // 子进程pid1用于发送
    pid1= fork();
    // 用于主进程阻塞
	while(1)
	{
        if(pid1==0)
        {
            //等待连接
    		clientaddr_len = sizeof(struct sockaddr_in);
            memset(&clientaddr, '\0', sizeof(struct sockaddr_in));
    		client_socket = accept(server_socket, (struct sockaddr*)(&clientaddr), &clientaddr_len);
    		if(client_socket == -1)
            {
                perror("The accept is error!\n");
                exit(0);
            }
            else
            {
                // 打印连接的子进程ip地址和端口信息
                char clntName[1024];
                if (inet_ntop(AF_INET, &clientaddr.sin_addr.s_addr, clntName, sizeof(clntName)) != NULL) {
                    printf("connecting client ip: %s port: %d\n", clntName, ntohs(clientaddr.sin_port));
                }
                else
                {
                    printf("client address get failed\n");
                }
            }
       		printf("client connected success!\n");
            // pid2 用于接收
            pid2=fork();
            if(pid2==0)
        	{
                // pid2 阻塞
                while (1)
                {
                    // 当子进程pid1检测到client断开连接后,及时退出子进程pid2,关闭socket
                    signal(SIGHUP, func_kill);
                    fgets(send_msg, sizeof(send_msg), stdin);
    			    send(client_socket, send_msg, sizeof(send_msg), 0);
                }
        	}else{
                // pid1 阻塞
                while (1)
                {	
                	int result = recv(client_socket,recv_msg, sizeof(recv_msg), 0);
        			// 子进程pid1检测到client断开连接
                    if (result <= 0)
                    {
                        printf("client connect close\n");
                        close(client_socket);
                        kill(pid2,SIGHUP);
                        break;
                    }
                    // 正常连接
                    printf("Client:%s",recv_msg);
                }
            }
        }
        else
        {
            // 阻塞父进程,pid1退出了就把父进程也退出
            int exit_pid = wait(&pid_exit_status);
            if (exit_pid == pid1)
            {
                // printf("paraent process quit!\n");
                close(server_socket);
                close(client_socket);  
                exit(0);
            }
        }
	}
	return 0;
}
```

## Client.c

```
#include <stdio.h>
#include <signal.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <sys/types.h>
#include <sys/wait.h>

#define PORT 23456
#define SERVER_ADDR "127.0.0.1"

int server_socket,client_socket;

void func_kill(int sig){
    close(server_socket);
    close(client_socket); 
    signal(SIGINT,SIG_DFL);
    exit(3);
}

int main(int argc, char *argv[])
{
    // 俩个子进程 pid1 用于接收 pid2 用于发送
	int pid1,pid2;
    // 等待pid退出信号
    int pid_exit_status;
    // 服务端地址
	struct sockaddr_in serveraddr;
	char recv_msg[100],send_msg[100];
	//创建socket
	server_socket = socket(AF_INET,SOCK_STREAM,0); 
	if(server_socket==-1)
	{
		perror("The socket is error!\n");
		exit(0);
	}
    // 设置端口绑定,取消TIME_WAIT,避免占用
    struct linger ling;
    ling.l_onoff = 1;
    ling.l_linger = 0;
    setsockopt(server_socket,SOL_SOCKET, SO_LINGER, &ling, sizeof(ling));

    // 设置服务端地址
	serveraddr.sin_family = AF_INET;
	serveraddr.sin_port = htons(PORT); 
    inet_aton(SERVER_ADDR,&serveraddr.sin_addr);
    bzero(serveraddr.sin_zero,8);

	//连接socket
	client_socket  = connect(server_socket, (struct sockaddr *)(&serveraddr),sizeof(serveraddr));
	if(client_socket==-1)
	{
		perror("The connect is error!\n");
		exit(0);
	}
    printf("server connect success！\n");

    // 捕获 ctrl+c信号
    signal(SIGINT,func_kill);

    // 子进程pid1用于发送
    pid1=fork();
    // 主进程阻塞
	while(1)
	{
        if (pid1==0)
        {  
            // 子进程pid2 用于发送
            pid2=fork();
            if(pid2==0)
        	{
                // pid2阻塞
                while (1)
                {
                    signal(SIGHUP,func_kill);
                    fgets(send_msg,sizeof(send_msg),stdin);
    			    send(server_socket,send_msg,sizeof(send_msg),0);
                }
        	}
            else
            {
                // pid1阻塞
                while (1)
                {
                    int result = recv(server_socket,recv_msg,sizeof(recv_msg),0);
            		if (result<=0)
                    {
                        printf("server close\n");
                        close(server_socket);
                        close(client_socket); 
                        // 与服务器断开连接,通知pid2退出,关闭socket
                        kill(pid2,SIGHUP);
                        exit(1);
                    }
                    printf("Server:%s",recv_msg);
                }
            }
        }
        else
        {
            // 阻塞父进程,pid1退出了就把父进程也退出
            int exit_pid = wait(&pid_exit_status);
            if (exit_pid==pid1)
            {
                // printf("paraent process quit!\n");
                close(server_socket);
                close(client_socket);  
                exit(0);
            }
        }
	}
	return 0;
}
```

## 运行结果

[![image-20201105201748025](https://blogjpg.yanmy.top/blog/20201105201755.png)](https://blogjpg.yanmy.top/blog/20201105201755.png)