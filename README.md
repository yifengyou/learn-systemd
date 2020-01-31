# systemd详解

```
Systemctl是一个systemd工具，主要负责控制systemd系统和服务管理器。
Systemd是一个系统管理守护进程、工具和库的集合，用于取代System V初始进程。Systemd的功能是用于集中管理和配置类UNIX系统。
```

![20200128_160617_66](image/20200128_160617_66.png)

## 本仓库内容

1. systemd学习笔记

```
Something I hope you know before go into the coding~
First, please watch or star this repo, I'll be more happy if you follow me.
Bug report, questions and discussion are welcome, you can post an issue or pull a request.
```

## 相关站点

* GitBook:<https://yifengyou.gitbooks.io/learn-systemd/content/>
* GitHub:<https://github.com/yifengyou/learn-systemd/>
* GitPage:<https://yifengyou.github.io/learn-systemd/>
* Systemd官方站点 : <https://freedesktop.org/wiki/Software/systemd/>

## 目录

* [systemd历史](docs/systemd历史.md)
    * [init进程](docs/systemd历史/init进程.md)
    * [systemd进程](docs/systemd历史/systemd进程.md)
    * [systemd的优劣](docs/systemd历史/systemd的优劣.md)
* [systemd概述](docs/systemd概述.md)
* [systemd特点](docs/systemd特点.md)
* [systemd架构](docs/systemd架构.md)
* [systemd基本概念](docs/systemd基本概念.md)
    * [unit](docs/systemd基本概念/unit.md)
    * [target](docs/systemd基本概念/target.md)
* [systemd相关命令](docs/systemd相关命令.md)
    * [bootctl](docs/systemd相关命令/bootctl.md)
    * [timedatectl](docs/systemd相关命令/timedatectl.md)
    * [busctl](docs/systemd相关命令/busctl.md)
    * [hostnamectl](docs/systemd相关命令/hostnamectl.md)
    * [kernel-install](docs/systemd相关命令/kernel-install.md)
    * [localectl](docs/systemd相关命令/localectl.md)
    * [networkctl](docs/systemd相关命令/networkctl.md)
    * [loginctl](docs/systemd相关命令/loginctl.md)
    * [journalctl](docs/systemd相关命令/journalctl.md)
    * [systemd-analyze](docs/systemd相关命令/systemd-analyze.md)
    * [systemd-cat](docs/systemd相关命令/systemd-cat.md)
    * [systemd-cgls](docs/systemd相关命令/systemd-cgls.md)
    * [systemd-cgtop](docs/systemd相关命令/systemd-cgtop.md)
    * [systemd-delta](docs/systemd相关命令/systemd-delta.md)
    * [systemd-detect-virt](docs/systemd相关命令/systemd-detect-virt.md)
    * [systemd-mount](docs/systemd相关命令/systemd-mount.md)
    * [systemd-umount](docs/systemd相关命令/systemd-umount.md)
    * [systemd-path](docs/systemd相关命令/systemd-path.md)
    * [systemd-resolve](docs/systemd相关命令/systemd-resolve.md)
    * [systemd-run](docs/systemd相关命令/systemd-run.md)
    * [systemd-socket-activate](docs/systemd相关命令/systemd-socket-activate.md)
    * [systemd-stdio-bridge](docs/systemd相关命令/systemd-stdio-bridge.md)


## 参考

* <https://blog.csdn.net/weixin_30894389/article/details/95682927>



## 小结


---

![20200131_215534_65](image/20200131_215534_65.png)


---
