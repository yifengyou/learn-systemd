# target

* 启动计算机的时候，需要启动大量的 Unit。如果每一次启动，都要一一写明本次启动需要哪些 Unit，显然非常不方便。
* Systemd 的解决方案就是 Target。简单说，Target 就是一个 Unit 组，包含许多相关的 Unit 。传统的init启动模式里面，有 RunLevel 的概念，跟 Target 的作用很类似。不同的是，RunLevel 是互斥的，不可能多个 RunLevel 同时启动，但是**多个 Target 可以同时启动**。

## 查看Target命令

查看当前系统的所有 Target

```
$ systemctl list-unit-files --type=target
```

查看一个 Target 包含的所有 Unit

```
$ systemctl list-dependencies multi-user.target
```

查看启动时的默认 Target

```
$ systemctl get-default
```

设置启动时的默认 Target

```
$ sudo systemctl set-default multi-user.target
```

**切换 Target 时，默认不关闭前一个 Target 启动的进程，**

```
$ sudo systemctl isolate 命令改变这种行为，
```

关闭前一个 Target 里面所有不属于后一个 Target 的进程

```
$ sudo systemctl isolate multi-user.target
```

## Target 与 传统 RunLevel 的对应关系

| Traditional runleve | New target name     Symbolically linked to...                                                    |
| ------------------- | ------------------------------------------------------------------------------------------------ |
| Runlevel 0          | runlevel0.target -> poweroff.target	关闭系统。                                                   |
| Runlevel 1          | runlevel1.target -> rescue.target	单用户模式。                                                   |
| Runlevel 2          | runlevel2.target -> multi-user.target	用户定义/域特定运行级别。默认等同于 3。                    |
| Runlevel 4          | runlevel4.target -> multi-user.target                                                            |
| Runlevel 3          | runlevel3.target -> multi-user.target	多用户，非图形化。用户可以通过多个控制台或网络登录。       |
| Runlevel 5          | runlevel5.target -> graphical.target	多用户，图形化。通常为所有运行级别 3 的服务外加图形化登录。 |
| Runlevel 6          | runlevel6.target -> reboot.target	重启                                                           |
| emergency           | emergency.target			紧急 Shell                                                                    |










---
