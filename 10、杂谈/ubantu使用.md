面向加薪学习

```shell
apt-get install openssh-server #安装openssh服务
apt-get install ufw #安装防火墙管理软件
ufw enable #开启防火墙
ufw allow 22 #打开22端口
netstat -ntlp|grep 22  #查看22端口是否启用
```

vim /etc/pam.d/gdm-autologin

#auth required pam_succeed_if.so user != root quiet_succes

vim /etc/pam.d/gdm-password

#auth required pam_succeed_if.so user != root quiet_success
