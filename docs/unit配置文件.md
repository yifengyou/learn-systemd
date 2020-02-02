# unit配置文件

* 12中unit配置文件按区块划分，包括
  - [Unit] : 通常是配置文件的第一个区块，用来定义 Unit 的元数据，以及配置与其他 Unit 的关系
  - [Service] : 只有 Service 类型的 Unit 才有这个区块
  - [Install] : 通常是配置文件的最后一个区块，定义如何启动，以及是否开机启动
* 12中unit都必须包含unit
* target类型一般仅unit，部分包含install
* service类型肯定包含service，unit。**不一定**包含insall
* 关于unit各个配置区块参考: <https://www.freedesktop.org/software/systemd/man/systemd.unit.html>


## 12中unit配置常见文件

1. service.service : nfs-server.service sshd.service kdump.service
2. socket.socket : dbus.socket sshd.socket
3. device.device :
4. mount.mount : tmp.mount
5. automount.automount :
6. swap.swap :
7. target.target : multi-user.target  graphical.target  poweroff.target
8. path.path :
9. timer.timer :
10. slice.slice : user.slice
11. scope.scope :


单元文件是 ini 风格的纯文本文件。 封装了有关下列对象的信息： 服务(service)、套接字(socket)、设备(device)、挂载点(mount)、自动挂载点(automount)、 启动目标(target)、交换分区或交换文件(swap)、被监视的路径(path)、任务计划(timer)、 资源控制组(slice)、一组外部创建的进程(scope)。

## unit配置文件注意事项

1. 单元文件可以通过一个"实例名"参数从"模板文件"构造出来(这个过程叫"**实例化**")。

* "模板文件"(也称"模板单元"或"单元模板")是定义一系列同类型单元的基础。
* 模板文件的名称必须以 "@" 结尾(在类型后缀之前)。 通过实例化得到的单元，其完整的单元名称是在模板文件的类型后缀与 "@" 之间插入实例名称形成的。
* 在通过实例化得到的单元内部， 可以使用 "%i" 以及其他说明符来引用实例参数。
* 除了手册中列出的选项之外，单元文件还可以包含更多其他选项。 无法识别的选项不会中断单元文件的加载，但是 systemd 会输出一条警告日志。
* 如果选项或者小节的名字以 X- 开头， 那么 systemd 将会完全忽略它。 以 X- 开头的小节中的选项没必要再以 X- 开头， 因为整个小节都已经被忽略。 应用程序可以利用这个特性在单元文件中包含额外的信息。

如果想要给一个单元赋予别名，那么可以按照需求，在系统单元目录或用户单元目录中， 创建一个软链接(以别名作为文件名)，并将其指向该单元的单元文件。

例如 systemd-networkd.service 在安装时就通过 /usr/lib/systemd/system/dbus-org.freedesktop.network1.service 软链接创建了 dbus-org.freedesktop.network1.service 别名。

此外，还可以直接在单元文件的 [Install] 小节中使用 Alias= 创建别名。

注意，单元文件中设置的别名会随着单元的启用(enable)与禁用(disable)而生效和失效， 也就是别名软链接会随着单元的启用(enable)与禁用(disable)而创建与删除。

例如，因为 reboot.target 单元文件中含有 Alias=ctrl-alt-del.target 的设置，所以启用(enable)此单元之后，按下 CTRL+ALT+DEL 组合键将会导致启动该单元。单元的别名可以用于 enable, disable, start, stop, status, … 这些命令中，也可以用于 Wants=, Requires=, Before=, After=, … 这些依赖关系选项中。 但是**务必注意，不可将单元的别名用于 preset 命令中**。 再次提醒，通过** Alias= 设置的别名仅在单元被启用(enable)之后才会生效**。

对于例如 foo.service 这样的单元文件， 可以同时存在对应的 foo.service.wants/ 与 foo.service.requires/ 目录， 其中可以放置许多指向其他单元文件的软链接。 软链接所指向的单元将会被隐含的添加到 foo.service 相应的 Wants= 与 Requires= 依赖中。 这样就可以方便的为单元添加依赖关系，而无需修改单元文件本身。 向 .wants/ 与 .requires/ 目录中添加软链接的首选方法是使用 systemctl 的 enable 命令， 它会读取单元文件的 [Install] 小节。

对于例如 foo.service 这样的单元文件，可以同时存在对应的 foo.service.d/ 目录， 当解析完主单元文件之后，目录中所有以 ".conf" 结尾的文件，都会被按照文件名的字典顺序，依次解析(相当于依次附加到主单元文件的末尾)。 这样就可以方便的修改单元的设置，或者为单元添加额外的设置，而无需修改单元文件本身。 注意，配置片段(".conf" 文件)必须包含明确的小节头(例如 "[Service]" 之类)。

对于从模板文件实例化而来的单元，会优先读取与此实例对应的 ".d/" 目录(例如 "foo@bar.service.d/")中的配置片段(".conf" 文件)， 然后才会读取与模板对应的 ".d/" 目录(例如 "foo@.service.d/")中的配置片段(".conf" 文件)。 对于名称中包含连字符("-")的单元，将会按特定顺序依次在一组(而不是一个)目录中搜索单元配置片段。

例如对于 foo-bar-baz.service 单元来说，将会依次在 foo-.service.d/, foo-bar-.service.d/, foo-bar-baz.service.d/ 目录下搜索单元配置片段。这个机制可以方便的为一组相关单元(单元名称的前缀都相同)定义共同的单元配置片段， 特别适合应用于 mount, automount, slice 类型的单元， 因为这些单元的命名规则就是基于连字符构建的。 注意，在前缀层次结构的下层目录中的单元配置片段，会覆盖上层目录中的同名文件， 也就是 foo-bar-.service.d/10-override.conf 会覆盖(取代) foo-.service.d/10-override.conf 文件。

存放配置片段(".conf" 文件)的 ".d/" 目录， 除了可以放置在 /etc/systemd/{system,user} 目录中， 还可以放置在 /usr/lib/systemd/{system,user} 与 /run/systemd/{system,user} 目录中。 虽然在优先级上，/etc 中的配置片段优先级最高、/run 中的配置片段优先级居中、 /usr/lib 中的配置片段优先级最低。但是这仅对同名配置片段之间的覆盖关系有意义。 因为所有 ".d/" 目录中的配置片段，无论其位于哪个目录， 都会被按照文件名的字典顺序，依次覆盖单元文件中的设置(相当于依次附加到主单元文件的末尾)。

注意，虽然 systemd 为明确设置单元之间的依赖关系提供了灵活的方法， 但是我们反对使用这些方法，你应该仅在万不得已的时候才使用它。 我们鼓励你使用基于 D-Bus 或 socket 的启动机制， 以将单元之间的依赖关系隐含化， 从而得到一个更简单也更健壮的系统。

如上所述，单元可以从模板实例化而来。 这样就可以用同一个模板文件衍生出多个单元。 当 systemd 查找单元文件时，会首先查找与单元名称完全吻合的单元文件， 如果没有找到，并且单元名称中包含 "@" 字符， 那么 systemd 将会继续查找拥有相同前缀的模板文件， 如果找到，那么将从这个模板文件实例化一个单元来使用。 例如，对于 getty@tty3.service 单元来说， 其对应的模板文件是 getty@.service (也就是去掉 "@" 与后缀名之间的部分)。

可以在模板文件内部 通过 "%i" 引用实例字符串(也就是上例中的"tty3")。

如果一个单元文件的大小为零字节或者是指向 /dev/null 的软链接， 那么它的所有相关配置都将被忽略。同时，该单元将被标记为 "masked" 状态，并且无法被启动。 这样就可以 彻底屏蔽一个单元(即使手动启动也不行)。
