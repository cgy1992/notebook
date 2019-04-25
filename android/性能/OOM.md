 使用 adb命令查看当前进程的线程数：

 adb shell

ps 查找当前 pid

cat /proc/pid/status

打印出来的信息中 threads 就是当前进程中包含的线程数
