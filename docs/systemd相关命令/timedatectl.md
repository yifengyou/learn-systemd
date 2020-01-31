# timedatectl

timedatectl命令用于查看当前时区设置

## 查看当前时区设置

```
$ timedatectl
```

![20200131_213437_39](image/20200131_213437_39.png)

## 显示所有可用的时区
```
$ timedatectl list-timezones
```

![20200131_213540_29](image/20200131_213540_29.png) 

## 设置当前时区

```
$ sudo timedatectl set-timezone America/New_York
$ sudo timedatectl set-time YYYY-MM-DD
$ sudo timedatectl set-time HH:MM:SS
```
