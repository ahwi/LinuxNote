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







































