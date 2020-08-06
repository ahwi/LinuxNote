# Tool - 关于Linux的工具使用



## cron 定时启动任务

如果需要定期或以预定的时间间隔执行任务，可以使用cron命令cron 是一个守护进程，可让你安排这些任务，然后按指定的时间间隔执行这些任务。



### 管理cron任务

1. 创建cron任务:

   以root用户创建或编辑cron任务:

   ```
   $ crontab -e
   ```



### 查看日志

```
$ cat /var/log/cron
```



### 参考:

<https://mp.weixin.qq.com/s/C48hmXSdGPwUBPDkhlrjQQ>



## sftp搭建

参考路径

https://tecadmin.net/create-sftp-only-user-centos/ 安装
https://www.poftut.com/what-is-default-sftp-port-and-how-to-change-sftp-port-number/ 修改端口号

* 创建用户

  ```
  sudo adduser --shell /bin/false sftpuser
  sudo passwd sftpuser
  ```

* 创建目录

  ```
  sudo mkdir -p /var/sftp/files
  sudo chown sftpuser:sftpuser /var/sftp/files
  sudo chown root:root /var/sftp
  sudo chmod 755 /var/sftp
  ```

* 配置ssh

  ```
  $ sudo vim /etc/ssh/sshd_config
  Match User sftpuser
  	ForceCommand internal-sftp
  	PasswordAuthentication yes
  	ChrootDirectory /var/sftp
  	PermitTunnel no
  	AllowAgentForwarding no
  	AllowTcpForwarding no
  	X11Forwarding no
  ```

* 重启ssh服务

  ```
  sudo systemctl restart sshd.service
  ```

* 如果权限有问题，可能需要修改权限

  ```
  如果出现连接出错，修改权限
  vi /etc/pam.d/sshd
  ```



































