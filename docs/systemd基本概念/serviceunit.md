# service unit

systemd.service — 服务单元配置


## 描述

以 ".service" 为后缀的单元文件， 封装了一个被 systemd 监视与控制的进程。

本手册列出了所有专用于此类单元的 配置选项(亦称"配置指令"或"单元属性")。 systemd.unit(5) 中描述了通用于所有单元类型的配置选项， 它们位于 "[Unit]" 与 "[Install]" 小节。 此类单元专用的配置选项 位于 "[Service]" 小节。

其他可用的选项参见 systemd.exec(5) 手册(定义了命令的执行环境)， 以及 systemd.kill(5) 手册(定义了如何结束进程)， 以及 systemd.resource-control(5) 手册(定义了进程的 资源控制)。

如果要求启动或停止的某个单元文件 不存在， systemd 将会寻找同名的SysV初始化脚本(去掉 .service 后缀)， 并根据那个同名脚本， 动态的创建一个 service 单元。 这主要用于与传统的SysV兼容(不能保证100%兼容)。 更多与SysV的兼容性可参见 Incompatibilities with SysV 文档。

服务模板
systemd 服务可以通过 "service@argument.service" 语法接受一个单独的参数。这样的服务被称为"实例化"服务，而不带 argument 参数的服务单元定义被称为"模板"。例如 dhcpcd@.service 服务模板接受一个网络接口参数之后， 就生成了一个实例化服务。在服务单元模板文件内部， 可以通过 %-specifiers 引用参数(也称为"实例名")。详见 systemd.unit(5) 手册。

自动依赖
隐含依赖
下列依赖关系是自动隐含的：

设置了 Type=dbus 的服务会自动添加 Requires=dbus.socket 与 After=dbus.socket 依赖。

基于套接字启动的服务会自动添加对关联的 .socket 单元的 After= 依赖。 服务单元还会为所有在 Sockets= 中列出的 .socket 单元自动添加 Wants= 与 After= 依赖。

还有一些 其他依赖关系是由 systemd.exec(5) 与 systemd.resource-control(5) 中的某些资源限制选项自动隐含添加的。

默认依赖
除非明确设置了 DefaultDependencies=no ，否则 service 单元将会自动添加下列依赖关系：

Requires=sysinit.target, After=sysinit.target, After=basic.target, Conflicts=shutdown.target, Before=shutdown.target 。 这样可以确保普通的服务单元： (1)在基础系统启动完毕之后才开始启动，(2)在关闭系统之前先被干净的停止。 只有那些需要在系统启动的早期就必须启动的服务， 以及那些必须在关机动作的结尾才能停止的服务才需要设置 DefaultDependencies=no 。

从同一个模版实例化出来的所有服务单元(单元名称中带有 "@" 字符)， 默认全部属于与模版同名的同一个 slice 单元(参见 systemd.slice(5))。 该同名 slice 一般在系统关机时，与所有模版实例一起停止。 如果你不希望像上面这样，那么可以在模版单元中明确设置 DefaultDependencies=no ， 并且：要么在该模版文件中明确定义特定的 slice 单元(同样也要明确设置 DefaultDependencies=no)、 要么在该模版文件中明确设置 Slice=system.slice (或其他合适的 slice)。 参见 systemd.resource-control(5) 手册。

选项
每个服务单元文件都必须包含一个 [Service] 小节。 由于此小节中的许多选项也 同时适用于其他类型的单元， 所以本手册仅记录了专用于服务单元的选项。 其他共享的选项参见 systemd.exec(5), systemd.kill(5), systemd.resource-control(5) 手册。 这里只列出仅能用于 [Service] 小节的 选项(亦称"指令"或"属性")：

Type=
设置进程的启动类型。必须设为 simple, exec, forking, oneshot, dbus, notify, idle 之一：

如果设为 simple (当设置了 ExecStart= 、 但是没有设置 Type= 与 BusName= 时，这是默认值)， 那么 ExecStart= 进程就是该服务的主进程， 并且 systemd 会认为在创建了该服务的主服务进程之后，该服务就已经启动完成。 如果此进程需要为系统中的其他进程提供服务， 那么必须在该服务启动之前先建立好通信渠道(例如套接字)， 这样，在创建主服务进程之后、执行主服务进程之前，即可启动后继单元， 从而加快了后继单元的启动速度。 这就意味着对于 simple 类型的服务来说， 即使不能成功调用主服务进程(例如 User= 不存在、或者二进制可执行文件不存在)， systemctl start 也仍然会执行成功。

exec 与 simple 类似，不同之处在于， 只有在该服务的主服务进程执行完成之后，systemd 才会认为该服务启动完成。 其他后继单元必须一直阻塞到这个时间点之后才能继续启动。换句话说， simple 表示当 fork() 函数返回时，即算是启动完成，而 exec 则表示仅在 fork() 与 execve() 函数都执行成功时，才算是启动完成。 这就意味着对于 exec 类型的服务来说， 如果不能成功调用主服务进程(例如 User= 不存在、或者二进制可执行文件不存在)， 那么 systemctl start 将会执行失败。

如果设为 forking ，那么表示 ExecStart= 进程将会在启动过程中使用 fork() 系统调用。 也就是当所有通信渠道都已建好、启动亦已成功之后，父进程将会退出，而子进程将作为主服务进程继续运行。 这是传统UNIX守护进程的经典做法。 在这种情况下，systemd 会认为在父进程退出之后，该服务就已经启动完成。 如果使用了此种类型，那么建议同时设置 PIDFile= 选项，以帮助 systemd 准确可靠的定位该服务的主进程。 systemd 将会在父进程退出之后 立即开始启动后继单元。

oneshot 与 simple 类似，不同之处在于， 只有在该服务的主服务进程退出之后，systemd 才会认为该服务启动完成，才会开始启动后继单元。 此种类型的服务通常需要设置 RemainAfterExit= 选项。 当 Type= 与 ExecStart= 都没有设置时， Type=oneshot 就是默认值。

dbus 与 simple 类似，不同之处在于， 该服务只有获得了 BusName= 指定的 D-Bus 名称之后，systemd 才会认为该服务启动完成，才会开始启动后继单元。 设为此类型相当于隐含的依赖于 dbus.socket 单元。 当设置了 BusName= 时， 此类型就是默认值。

notify 与 exec 类似，不同之处在于， 该服务将会在启动完成之后通过 sd_notify(3) 之类的接口发送一个通知消息。systemd 将会在启动后继单元之前， 首先确保该进程已经成功的发送了这个消息。如果设为此类型，那么下文的 NotifyAccess= 将只能设为非 none 值。如果未设置 NotifyAccess= 选项、或者已经被明确设为 none ，那么将会被自动强制修改为 main 。注意，目前 Type=notify 尚不能与 PrivateNetwork=yes 一起使用。

idle 与 simple 类似，不同之处在于， 服务进程将会被延迟到所有活动任务都完成之后再执行。 这样可以避免控制台上的状态信息与shell脚本的输出混杂在一起。 注意：(1)仅可用于改善控制台输出，切勿将其用于不同单元之间的排序工具； (2)延迟最多不超过5秒， 超时后将无条件的启动服务进程。

建议对长时间持续运行的服务尽可能使用 Type=simple (这是最简单和速度最快的选择)。 注意，因为 simple 类型的服务 无法报告启动失败、也无法在服务完成初始化后对其他单元进行排序， 所以，当客户端需要通过仅由该服务本身创建的IPC通道(而非由 systemd 创建的套接字或 D-bus 之类)连接到该服务的时候， simple 类型并不是最佳选择。在这种情况下， notify 或 dbus(该服务必须提供 D-Bus 接口) 才是最佳选择， 因为这两种类型都允许服务进程精确的安排 何时算是服务启动成功、何时可以继续启动后继单元。 notify 类型需要服务进程明确使用 sd_notify() 函数或类似的API， 否则，可以使用 forking 作为替代(它支持传统的UNIX服务启动协议)。 最后，如果能够确保服务进程调用成功、服务进程自身不做或只做很少的初始化工作(且不大可能初始化失败)， 那么 exec 将是最佳选择。 注意，因为使用任何 simple 之外的类型都需要等待服务完成初始化，所以可能会减慢系统启动速度。 因此，应该尽可能避免使用 simple 之外的类型(除非必须)。另外，也不建议对长时间持续运行的服务使用 idle 或 oneshot 类型。

RemainAfterExit=
当该服务的所有进程全部退出之后， 是否依然将此服务视为活动(active)状态。 默认值为 no

GuessMainPID=
在无法明确定位 该服务主进程的情况下， systemd 是否应该猜测主进程的PID(可能不正确)。 该选项仅在设置了 Type=forking 但未设置 PIDFile= 的情况下有意义。 如果PID猜测错误， 那么该服务的失败检测与自动重启功能将失效。 默认值为 yes

PIDFile=
该服务PID文件的路径(一般位于 /run/ 目录下)。 强烈建议在 Type=forking 的情况下明确设置此选项。 如果设为相对路径，那么表示相对于 /run/ 目录。 systemd 将会在此服务启动完成之后，从此文件中读取主服务进程的PID 。 systemd 不会写入此文件，但会在此服务停止后删除它(若仍然存在)。 PID文件的拥有者不必是特权用户， 但是如果拥有者是非特权用户，那么必须施加如下安全限制： (1)不能是一个指向其他拥有者文件的软连接(无论直接还是间接)； (2)其中的PID必须指向一个属于该服务的进程。

BusName=
设置与此服务通信 所使用的 D-Bus 名称。 在 Type=dbus 的情况下 必须明确设置此选项。

ExecStart=
在启动该服务时需要执行的 命令行(命令+参数)。 有关命令行的更多细节， 可参见后文的"命令行"小节。

除非 Type=oneshot ，否则必须且只能设置一个命令行。 仅在 Type=oneshot 的情况下，才可以设置任意个命令行(包括零个)， 多个命令行既可以在同一个 ExecStart= 中设置，也可以通过设置多个 ExecStart= 来达到相同的效果。 如果设为一个空字符串，那么先前设置的所有命令行都将被清空。 如果不设置任何 ExecStart= 指令， 那么必须确保设置了 RemainAfterExit=yes 指令，并且至少设置一个 ExecStop= 指令。 同时缺少 ExecStart= 与 ExecStop= 的服务单元是非法的(也就是必须至少明确设置其中之一)。

命令行必须以一个可执行文件(要么是绝对路径、要么是不含任何斜线的文件名)开始， 并且其后的那些参数将依次作为"argv[1] argv[2] …"传递给被执行的进程。 可选的，可以在绝对路径前面加上各种不同的前缀表示不同的含义：

表 1. 可执行文件前的特殊前缀

前缀	效果
"@"	如果在绝对路径前加上可选的 "@" 前缀，那么其后的那些参数将依次作为"argv[0] argv[1] argv[2] …"传递给被执行的进程(注意，argv[0] 是可执行文件本身)。
"-"	如果在绝对路径前加上可选的 "-" 前缀，那么即使该进程以失败状态(例如非零的返回值或者出现异常)退出，也会被视为成功退出，但同时会留下错误日志。
"+"	如果在绝对路径前加上可选的 "+" 前缀，那么进程将拥有完全的权限(超级用户的特权)，并且 User=, Group=, CapabilityBoundingSet= 选项所设置的权限限制以及 PrivateDevices=, PrivateTmp= 等文件系统名字空间的配置将被该命令行启动的进程忽略(但仍然对其他 ExecStart=, ExecStop= 有效)。
"!"	与 "+" 类似(进程仍然拥有超级用户的身份)，不同之处在于仅忽略 User=, Group=, SupplementaryGroups= 选项的设置，而例如名字空间之类的其他限制依然有效。注意，当与 DynamicUser= 一起使用时，将会在执行该命令之前先动态分配一对 user/group ，然后将身份凭证的切换操作留给进程自己去执行。
"!!"	与 "!" 极其相似，仅用于让利用 ambient capability 限制进程权限的单元兼容不支持 ambient capability 的系统(也就是不支持 AmbientCapabilities= 选项)。如果在不支持 ambient capability 的系统上使用此前缀，那么 SystemCallFilter= 与 CapabilityBoundingSet= 将被隐含的自动修改为允许进程自己丢弃 capability 与特权用户的身份(即使原来被配置为禁止这么做)，并且 AmbientCapabilities= 选项将会被忽略。此前缀在支持 ambient capability 的系统上完全没有任何效果。

"@", "-" 以及 "+"/"!"/"!!" 之一，可以按任意顺序同时混合使用。 注意，对于 "+", "!", "!!" 前缀来说，仅能单独使用三者之一，不可混合使用多个。 注意，这些前缀同样也可以用于 ExecStartPre=, ExecStartPost=, ExecReload=, ExecStop=, ExecStopPost= 这些接受命令行的选项。

如果设置了多个命令行， 那么这些命令行将以其在单元文件中出现的顺序依次执行。 如果某个无 "-" 前缀的命令行执行失败， 那么剩余的命令行将不会被继续执行， 同时该单元将变为失败(failed)状态。

当未设置 Type=forking 时， 这里设置的命令行所启动的进程 将被视为该服务的主守护进程。

ExecStartPre=, ExecStartPost=
设置在执行 ExecStart= 之前/后执行的命令行。 语法规则与 ExecStart= 完全相同。 如果设置了多个命令行， 那么这些命令行将以其在单元文件中出现的顺序 依次执行。

如果某个无 "-" 前缀的命令行执行失败， 那么剩余的命令行将不会被继续执行， 同时该单元将变为失败(failed)状态。

仅在所有无 "-" 前缀的 ExecStartPre= 命令全部执行成功的前提下， 才会继续执行 ExecStart= 命令。

ExecStartPost= 命令仅在 ExecStart= 中的命令已经全部执行成功之后才会运行， 判断的标准基于 Type= 选项。 具体说来，对于 Type=simple 或 Type=idle 就是主进程已经成功启动； 对于 Type=oneshot 来说就是最后一个 ExecStart= 进程已经成功退出； 对于 Type=forking 来说就是初始进程已经成功退出； 对于 Type=notify 来说就是已经发送了 "READY=1" ； 对于 Type=dbus 来说就是已经取得了 BusName= 中设置的总线名称。

注意，不可将 ExecStartPre= 用于 需要长时间执行的进程。 因为所有由 ExecStartPre= 派生的子进程 都会在启动 ExecStart= 服务进程之前被杀死。

注意，如果在服务启动完成之前，任意一个 ExecStartPre=, ExecStart=, ExecStartPost= 中无 "-" 前缀的命令执行失败或超时， 那么，ExecStopPost= 将会被继续执行，而 ExecStop= 则会被跳过。

ExecReload=
这是一个可选的指令， 用于设置当该服务 被要求重新载入配置时 所执行的命令行。 语法规则与 ExecStart= 完全相同。

另外，还有一个特殊的环境变量 $MAINPID 可用于表示主进程的PID， 例如可以这样使用：

/bin/kill -HUP $MAINPID
注意，像上例那样，通过向守护进程发送复位信号， 强制其重新加载配置文件，并不是一个好习惯。 因为这是一个异步操作， 所以不适用于需要按照特定顺序重新加载配置文件的服务。 我们强烈建议将 ExecReload= 设为一个 能够确保重新加载配置文件的操作同步完成的命令行。

ExecStop=
这是一个可选的指令， 用于设置当该服务被要求停止时所执行的命令行。 语法规则与 ExecStart= 完全相同。 执行完此处设置的所有命令行之后，该服务将被视为已经停止， 此时，该服务所有剩余的进程将会根据 KillMode= 的设置被杀死(参见 systemd.kill(5))。 如果未设置此选项，那么当此服务被停止时， 该服务的所有进程都将会根据 KillSignal= 的设置被立即全部杀死。 与 ExecReload= 一样， 也有一个特殊的环境变量 $MAINPID 可用于表示主进程的PID 。

一般来说，不应该仅仅设置一个结束服务的命令而不等待其完成。 因为当此处设置的命令执行完之后， 剩余的进程会被按照 KillMode= 与 KillSignal= 的设置立即杀死， 这可能会导致数据丢失。 因此，这里设置的命令必须是同步操作，而不能是异步操作。

注意，仅在服务确实启动成功的前提下，才会执行 ExecStop= 中设置的命令。 如果服务从未启动或启动失败(例如，任意一个 ExecStart=, ExecStartPre=, ExecStartPost= 中无 "-" 前缀的命令执行失败或超时)， 那么 ExecStop= 将会被跳过。 如果想要无条件的在服务停止后执行特定的动作，那么应该使用 ExecStopPost= 选项。 如果服务启动成功，那么即使主服务进程已经终止(无论是主动退出还是被杀死)，也会继续执行停止操作。 因此停止命令必须正确处理这种场景，如果 systemd 发现在调用停止命令时主服务进程已经终止，那么将会撤销 $MAINPID 变量。

重启服务的动作被实现为"先停止、再启动"。所以在重启期间，将会执行 ExecStop= 与 ExecStopPost= 命令。 推荐将此选项用于那些必须在服务干净退出之前执行的命令(例如还需要继续与主服务进程通信)。当此选项设置的命令被执行的时候，应该假定服务正处于完全正常的运行状态，可以正常的与其通信。 如果想要无条件的在服务停止后"清理尸体"，那么应该使用 ExecStopPost= 选项。

ExecStopPost=
这是一个可选的指令， 用于设置在该服务停止之后所执行的命令行。 语法规则与 ExecStart= 完全相同。 注意，与 ExecStop= 不同，无论服务是否启动成功， 此选项中设置的命令都会在服务停止后被无条件的执行。

应该将此选项用于设置那些无论服务是否启动成功， 都必须在服务停止后无条件执行的清理操作。 此选项设置的命令必须能够正确处理由于服务启动失败而造成的各种残缺不全以及数据不一致的场景。 由于此选项设置的命令在执行时，整个服务的所有进程都已经全部结束， 所以无法与服务进行任何通信。

注意，此处设置的所有命令在被调用之后都可以读取如下环境变量： $SERVICE_RESULT(服务的最终结果), $EXIT_CODE(服务主进程的退出码), $EXIT_STATUS(服务主进程的退出状态)。 详见 systemd.exec(5) 手册。

RestartSec=
设置在重启服务(Restart=)前暂停多长时间。 默认值是100毫秒(100ms)。 如果未指定时间单位，那么将视为以秒为单位。 例如设为"20"等价于设为"20s"。

TimeoutStartSec=
设置该服务允许的最大启动时长。 如果守护进程未能在限定的时长内发出"启动完毕"的信号，那么该服务将被视为启动失败，并会被关闭。 如果未指定时间单位，那么将视为以秒为单位。 例如设为"20"等价于设为"20s"。 设为 "infinity" 则表示永不超时。 当 Type=oneshot 时， 默认值为 "infinity" (永不超时)， 否则默认值等于 DefaultTimeoutStartSec= 的值(参见 systemd-system.conf(5) 手册)。

如果一个 Type=notify 服务发送了 "EXTEND_TIMEOUT_USEC=…" 信号， 那么允许的启动时长将会在 TimeoutStartSec= 基础上继续延长指定的时长。 注意，必须在 TimeoutStartSec= 用完之前发出第一个延时信号。当启动时间超出 TimeoutStartSec= 之后，该服务可以在维持始终不超时的前提下，不断重复发送 "EXTEND_TIMEOUT_USEC=…" 信号， 直到完成启动(发送 "READY=1" 信号)。详见 sd_notify(3) 手册。

TimeoutStopSec=
此选项有两个用途： (1)设置每个 ExecStop= 的超时时长。如果其中之一超时， 那么所有后继的 ExecStop= 都会被取消，并且该服务也会被 SIGTERM 信号强制关闭。 如果该服务没有设置 ExecStop= ，那么该服务将会立即被 SIGTERM 信号强制关闭。 (2)设置该服务自身停止的超时时长。如果超时，那么该服务将会立即被 SIGTERM 信号强制关闭(参见 systemd.kill(5) 手册中的 KillMode= 选项)。 如果未指定时间单位，那么将视为以秒为单位。 例如设为"20"等价于设为"20s"。 设为 "infinity" 则表示永不超时。 默认值等于 DefaultTimeoutStopSec= 的值(参见 systemd-system.conf(5) 手册)。

如果一个 Type=notify 服务发送了 "EXTEND_TIMEOUT_USEC=…" 信号， 那么允许的停止时长将会在 TimeoutStopSec= 基础上继续延长指定的时长。 注意，必须在 TimeoutStopSec= 用完之前发出第一个延时信号。当停止时间超出 TimeoutStopSec= 之后，该服务可以在维持始终不超时的前提下，不断重复发送 "EXTEND_TIMEOUT_USEC=…" 信号，直到完成停止。 详见 sd_notify(3) 手册。

TimeoutSec=
一个同时设置 TimeoutStartSec= 与 TimeoutStopSec= 的快捷方式。

RuntimeMaxSec=
允许服务持续运行的最大时长。 如果服务持续运行超过了此处限制的时长，那么该服务将会被强制终止，同时将该服务变为失败(failed)状态。 注意，此选项对 Type=oneshot 类型的服务无效，因为它们会在启动完成后立即终止。 默认值为 "infinity" (不限时长)。

如果一个 Type=notify 服务发送了 "EXTEND_TIMEOUT_USEC=…" 信号， 那么允许的运行时长将会在 RuntimeMaxSec= 基础上继续延长指定的时长。 注意，必须在 RuntimeMaxSec= 用完之前发出第一个延时信号。当运行时间超出 RuntimeMaxSec= 之后，该服务可以在维持始终不超时的前提下，不断重复发送 "EXTEND_TIMEOUT_USEC=…" 信号， 直到运行结束(发送 "STOPPING=1" 信号或直接退出)。详见 sd_notify(3) 手册。

WatchdogSec=
设置该服务的看门狗(watchdog)的超时时长。 看门狗将在服务成功启动之后被启动。 该服务在运行过程中必须周期性的以 "WATCHDOG=1" ("keep-alive ping")调用 sd_notify(3) 函数。 如果在两次调用之间的时间间隔大于这里设定的值， 那么该服务将被视为失败(failed)状态， 并会被强制使用 WatchdogSignal= 信号(默认为 SIGABRT)关闭。 通过将 Restart= 设为 on-failure, on-watchdog, on-abnormal, always 之一， 可以实现在失败状态下的自动重启该服务。 这里设置的值将会通过 WATCHDOG_USEC= 环境变量传递给守护进程， 这样就允许那些支持看门狗的服务自动启用"keep-alive ping"。 如果设置了此选项， 那么 NotifyAccess= 将只能设为非 none 值。 如果 NotifyAccess= 未设置，或者已经被明确设为 none ， 那么将会被自动强制修改为 main 。 如果未指定时间单位，那么将视为以秒为单位。 例如设为"20"等价于设为"20s"。 默认值"0"表示禁用看门狗功能。 详见 sd_watchdog_enabled(3) 与 sd_event_set_watchdog(3) 手册。

Restart=
当服务进程 正常退出、异常退出、被杀死、超时的时候， 是否重新启动该服务。 所谓"服务进程" 是指 ExecStartPre=, ExecStartPost=, ExecStop=, ExecStopPost=, ExecReload= 中设置的进程。 当进程是由于 systemd 的正常操作(例如 systemctl stop|restart)而被停止时， 该服务不会被重新启动。 所谓"超时"可以是看门狗的"keep-alive ping"超时， 也可以是 systemctl start|reload|stop 操作超时。

该选项的值可以取 no, on-success, on-failure, on-abnormal, on-watchdog, on-abort, always 之一。 no(默认值) 表示不会被重启。 always 表示会被无条件的重启。 on-success 表示仅在服务进程正常退出时重启， 所谓"正常退出"是指：退出码为"0"， 或者进程收到 SIGHUP, SIGINT, SIGTERM, SIGPIPE 信号之一， 并且 退出码符合 SuccessExitStatus= 的设置。 on-failure 表示 仅在服务进程异常退出时重启， 所谓"异常退出" 是指： 退出码不为"0"， 或者 进程被强制杀死(包括 "core dump"以及收到 SIGHUP, SIGINT, SIGTERM, SIGPIPE 之外的其他信号)， 或者进程由于 看门狗超时 或者 systemd 的操作超时 而被杀死。

表 2. Restart= 的设置分别对应于哪些退出原因

退出原因(↓) | Restart= (→)	no	always	on-success	on-failure	on-abnormal	on-abort	on-watchdog
正常退出	 	X	X	 	 	 	 
退出码不为"0"	 	X	 	X	 	 	 
进程被强制杀死	 	X	 	X	X	X	 
systemd 操作超时	 	X	 	X	X	 	 
看门狗超时	 	X	 	X	X	 	X

注意如下例外情况(详见下文)： (1) RestartPreventExitStatus= 中列出的退出码或信号永远不会导致该服务被重启。 (2) 被 systemctl stop 命令或等价的操作停止的服务永远不会被重启。 (3) RestartForceExitStatus= 中列出的退出码或信号将会 无条件的导致该服务被重启。

注意，服务的重启频率仍然会受到由 StartLimitIntervalSec= 与 StartLimitBurst= 定义的启动频率的制约。详见 systemd.unit(5) 手册。只有在达到启动频率限制之后， 重新启动的服务才会进入失败状态。

对于需要长期持续运行的守护进程， 推荐设为 on-failure 以增强可用性。 对于自身可以自主选择何时退出的服务， 推荐设为 on-abnormal

SuccessExitStatus=
额外定义其他的进程"正常退出"状态。 也就是，在退出码"0"、以及表示"正常退出"的 SIGHUP, SIGINT, SIGTERM, SIGPIPE 信号之外， 再额外添加一组表示"正常退出"的退出码或信号。 可以设为一系列 以空格分隔的数字退出码或者信号名称， 例如：

SuccessExitStatus=1 2 8 SIGKILL
表示当进程的退出码是 1, 2, 8 或被 SIGKILL 信号终止时， 都可以视为"正常退出"。

如果多次使用此选项， 那么最终的结果将是多个列表的合并。 如果将此选项设为空， 那么先前设置的列表 将被清空。

RestartPreventExitStatus=
可以设为一系列 以空格分隔的数字退出码或信号名称， 当进程的退出码或收到的信号与此处的设置匹配时， 无论 Restart= 选项 是如何设置的， 该服务都将无条件的 禁止重新启动。 例如：

RestartPreventExitStatus=1 6 SIGABRT
可以确保退出码 1, 6 与 SIGABRT 信号 不会导致该服务被自动重启。 默认值为空， 表示完全遵守 Restart= 的设置。 如果多次使用此选项，那么最终的结果将是多个列表的合并。 如果将此选项设为空，那么先前设置的列表将被清空。

RestartForceExitStatus=
可以设为一系列以空格分隔的数字退出码或信号名称， 当进程的退出码或收到的信号与此处的设置匹配时， 无论 Restart= 是如何设置的，该服务都将无条件的被自动重新启动。 默认值为空，表示完全遵守 Restart= 的设置。 如果多次使用此选项，那么最终的结果将是多个列表的合并。 如果将此选项设为空，那么先前设置的列表将被清空。

RootDirectoryStartOnly=
接受一个布尔值。 设为 yes 表示根目录 RootDirectory= 选项(参见 systemd.exec(5) 手册) 仅对 ExecStart= 中的程序有效， 而对 ExecStartPre=, ExecStartPost=, ExecReload=, ExecStop=, ExecStopPost= 中的程序无效。 默认值 no 表示根目录对所有 Exec*= 系列选项中的程序都有效。

NonBlocking=
是否为所有基于套接字启动传递的文件描述符设置非阻塞标记(O_NONBLOCK)。 设为 yes 表示除了通过 FileDescriptorStoreMax= 引入的文件描述符之外， 所有 ≥3 的文件描述符(非 stdin, stdout, stderr 文件描述符)都将被设为非阻塞模式。 该选项仅在与 socket 单元 (systemd.socket(5)) 联用的时候才有意义。 对于那些先前已经通过 FileDescriptorStoreMax= 引入的文件描述符则毫无影响。 默认值为 no

NotifyAccess=
设置通过 sd_notify(3) 访问服务状态通知套接字的模式。 可以设为 none(默认值), main, exec, all 之一。 none 表示不更新任何守护进程的状态，忽略所有状态更新消息。 main 表示仅接受主进程的状态更新消息。 exec 表示仅接受主进程以及 Exec*= 进程的状态更新消息。 all 表示接受该服务cgroup内所有进程的状态更新消息。 当设置了 Type=notify 或 WatchdogSec= 的时候(见前文)，此选项将只能设为非 none 值。 如果 NotifyAccess= 未设置，或者已经被明确设为 none ， 那么将会被自动强制修改为 main 。

注意，服务单元的 sd_notify() 通知能够正常工作的前提， 是必须满足如下两个条件之一： (1)在 PID=1 的进程处理通知消息时，发送该通知的进程依然在运行； (2)发送该通知的进程是 systemd 派生的子进程(也就是匹配 main 或 exec 的进程)。 如果服务单元中的某个辅助进程在发送了 sd_notify() 通知之后就立即退出了， 那么 systemd 将有可能来不及将该通知关联到这个服务单元上。 在这种情况下，即使明确设置了 NotifyAccess=all ， 该通知也可能会被忽略掉。

Sockets=
设置一个 socket 单元的名称， 表示该服务在启动时应当从它继承套接字文件描述符。 通常并不需要明确设置此选项， 因为所有与该服务同名(不算后缀)的 socket 单元的套接字文件描述符， 都会被自动的 传递给派生进程。

注意： (1)同一个套接字文件描述符可以被传递给多个不同的进程(服务)。 (2)当套接字上有流量进入时， 被启动的可能是另一个不同于该服务的其他服务。 换句话说就是： 套接字单元中的 Sockets= 所指向的服务单元中的 Sockets= 未必要反向指回去。

如果多次使用此选项， 那么最终的结果将是多个socket单元的合集。 如果将此选项设为空， 那么先前设置的所有socket单元 都将被清空。

FileDescriptorStoreMax=
允许在 systemd 中最多为该服务存储多少个使用 "FDSTORE=1" 消息(sd_pid_notify_with_fds(3)) 的文件描述符。默认值为"0"(不存储)。 通过将服务重启过程中不应该关闭的套接字与文件描述符使用这种方法保存起来， 就可以实现让服务在重启(正常重启或崩溃重启)之后不丢失其状态。 进程的状态可以被序列化为一个文件之后保存在 /run 中， 或者保存在一个 memfd_create(2) 内存文件描述符中(这是更好的选择)。 所有被 systemd 暂存的文件描述符都将在该服务重启之后交还给该服务的主进程。 所有被 systemd 暂存的文件描述符都将在遇到如下两种情况时被自动关闭： (1)收到 POLLHUP 或 POLLERR 信号； (2)该服务被彻底停止，并且没有任何剩余的任务需要处理。 如果使用了此选项，那么前文的 NotifyAccess= 应该被设为允许访问 systemd 提供的通知套接字。若未设置 NotifyAccess= ，那么将被隐含的设为 main

USBFunctionDescriptors=
设为一个包含 USB FunctionFS 描述符的文件路径， 以实现 USB gadget 支持。 仅与配置了 ListenUSBFunction= 的 socket 单元一起使用。该文件的内容将被写入 ep0 文件。

USBFunctionStrings=
设为一个包含 USB FunctionFS 字符串的文件路径。 其行为与上面的 USBFunctionDescriptors= 类似。

参见 systemd.exec(5) 与 systemd.kill(5) 手册，以了解更多其他选项。

命令行
本小节 讲解 ExecStart=, ExecStartPre=, ExecStartPost=, ExecReload=, ExecStop=, ExecStopPost= 选项的命令行解析规则。

如果要一次设置多个命令，那么可以使用分号(;)将多个命令行连接起来。 注意，仅在设置了 Type=oneshot 的前提下，才可以一次设置多个命令。 分号自身 必须用 "\;" 表示。

每个命令行的内部 都以空格分隔， 第一项是要运行的命令， 随后的各项则是命令的参数。 每一项的边界都可以用单引号或双引号界定， 但引号自身最终将会被剥离。 还可以使用C语言风格的转义序列， 但仅可使用下文表格中的转义序列。 最后，行尾的反斜杠("\") 将被视作续行符(借鉴了bash续行语法)。

命令行的语法刻意借鉴了shell中的转义字符与变量展开语法， 但两者并不完全相同。 特别的， 重定向("<", "<<", ">", ">>")、 管道("|")、 后台运行("&")， 以及其他下文未明确提及的符号都不被支持。

要运行的命令(第一项)可以包含空格，但是不能包含控制字符。

可以在各项命令参数中使用 "%" 系列替换标记(详见 systemd.unit(5)手册)。

支持 "${FOO}" 与 "$FOO" 两种不同的环境变量替换方式。 具体说来就是： "${FOO}" 的内容将原封不动的转化为一个单独的命令行参数， 无论其中是否包含空格与引号，也无论它是否为空。 "$FOO" 的内容将被原封不动的插入命令行中， 但对插入内容的解释却遵守一般的命令行解析规则。 后文的两个例子， 将能清晰的体现两者的差别。

如果要运行的命令(第一项)不是一个绝对路径， 那么将会在编译时设定的可执行文件搜索目录中查找。 因为默认包括 /usr/local/bin/, /usr/bin/, /bin/, /usr/local/sbin/, /usr/sbin/, /sbin/ 目录， 所以可以安全的直接使用"标准目录"中的可执行程序名称(没必要再使用绝对路径)， 而对于非标准目录中的可执行程序，则必须使用绝对路径。建议始终使用绝对路径以避免歧义。 [提示]可以使用 systemd-path search-binaries-default 显示编译时设定的可执行文件搜索目录。

例(1)：

Environment="ONE=one" 'TWO=two two'
ExecStart=echo $ONE $TWO ${TWO}
这将给 /bin/echo 命令依次传递如下四个参数: "one", "two", "two", "two two"

例(2)：

Environment=ONE='one' "TWO='two two' too" THREE=
ExecStart=/bin/echo ${ONE} ${TWO} ${THREE}
ExecStart=/bin/echo $ONE $TWO $THREE
这将导致 /bin/echo 被执行两次。 第一次被依次传递如下三个参数： "'one'", "'two two' too", "" ； 第二次被依次传递如下三个参数： "one", "two two", "too" 。

此外，如果想要传递美元符号($)自身， 则必须使用 "$$" 。 而那些无法在替换时确定内容的变量将被当做空字符串。 注意，不可以在第一项(也就是命令的绝对路径)中使用变量替换。

注意，这里使用的变量必须已经在 Environment= 或 EnvironmentFile= 中定义。 此外，在 systemd.exec(5) 手册的"环境变量"小节中列出的"静态变量"也可以使用。 例如 $USER 就是一个"静态变量"， 而 $TERM 则不是。

注意， 这里的命令行并不直接支持shell命令， 但是可以通过模仿下面这个变通的方法来实现：

ExecStart=sh -c 'dmesg | tac'
例一

ExecStart=echo one ; echo "two two"
这将导致 echo 被执行两次。 第一次被传递了单独一个 "one" 参数； 第二次被传递了单独一个 "two two" 参数。 因为一次设置了多个命令，所以仅能用于 Type=oneshot 类型。

例二

ExecStart=echo / >/dev/null & \; \
ls
这表示向 echo 命令传递五个参数： "/", ">/dev/null", "&", ";", "ls"

表 3. 可以在命令行与环境变量中使用的C语言风格的转义序列

转义序列	实际含义
"\a"	响铃
"\b"	退格
"\f"	换页
"\n"	换行
"\r"	回车
"\t"	制表符
"\v"	纵向制表符
"\\"	反斜线
"\""	双引号
"\'"	单引号
"\s"	空白
"\xxx"	十六进制数 xx 所对应的字符
"\nnn"	八进制数 nnn 所对应的字符

例子
例 1. 简单服务

下面的单元文件创建了一个运行 /usr/sbin/foo-daemon 守护进程的服务。 未设置 Type= 等价于 Type=simple 默认设置。 systemd 执行守护进程之后， 即认为该单元已经启动成功。

[Unit]
Description=简单的Foo服务

[Service]
ExecStart=/usr/sbin/foo-daemon

[Install]
WantedBy=multi-user.target
注意，本例中的 /usr/sbin/foo-daemon 必须在启动后持续运行到服务被停止。 如果该进程只是为了派生守护进程，那么应该使用 Type=forking

因为没有设置 ExecStop= 选项， 所以在停止服务时，systemd 将会直接向该服务启动的所有进程发送 SIGTERM 信号。 若超过指定时间依然存在未被杀死的进程，那么将会继续发送 SIGKILL 信号。 详见 systemd.kill(5) 手册。

默认的 Type=simple 并不包含任何通知机制(例如通知"服务启动成功")。 要想使用通知机制，应该将 Type= 设为其他非默认值： Type=notify 可用于能够理解 systemd 通知协议的服务； Type=forking 可用于能将自身切换到后台的服务； Type=dbus 可用于能够在完成初始化之后 获得一个 D-Bus 名称的单元。


例 2. 一次性服务

Type=oneshot 用于那些只需要执行一次性动作而不需要持久运行的单元， 例如文件系统检查或者清理临时文件。 此类单元， 将会在启动后一直等待指定的动作完成， 然后再回到停止状态。 下面是一个执行清理动作的单元：

[Unit]
Description=清理老旧的 Foo 数据

[Service]
Type=oneshot
ExecStart=/usr/sbin/foo-cleanup

[Install]
WantedBy=multi-user.target
注意，在 /usr/sbin/foo-cleanup 执行结束前， 该服务一直处于"启动中"(activating)状态，而一旦执行结束，该服务又立即变为"停止"(inactive)状态。 也就是说，对于 Type=oneshot 类型的服务，不存在"活动"(active)状态。 这意味着，如果再一次启动该服务，将会再一次执行该服务定义的动作。 注意，在先后顺序上晚于该服务的单元， 将会一直等到该服务变成"停止"(inactive)状态后， 才会开始启动。

Type=oneshot 是唯一可以设置多个 ExecStart= 指令的服务类型。 多个 ExecStart= 指令将按照它们出现的顺序依次执行， 一旦遇到错误，就会立即停止，不再继续执行， 同时该服务也将进入"失败"(failed)状态。


例 3. 可停止的一次性服务

有时候， 单元需要执行一个程序以完成某个设置(启动)， 然后又需要再执行另一个程序以撤消先前的设置(停止)， 而在设置持续有效的时段中，该单元应该视为处于"活动"(active)状态， 但实际上并无任何程序在持续运行。 网络配置服务就是一个典型的例子。 此外，只能启动一次(不可多次启动)的一次性服务， 也是一个例子。

可以通过设置 RemainAfterExit=yes 来满足这种需求。 在这种情况下，systemd 将会在启动成功后将该单元视为处于"活动"(active)状态(而不是"停止"(inactive)状态)。 RemainAfterExit=yes 虽然可以用于所有 Type= 类型， 但是在实践中主要用于 Type=oneshot 和 Type=simple 类型。 对于 Type=oneshot 类型， systemd 一直等到服务启动成功之后，才会将该服务置于"活动"(active)状态。 所以，依赖于该服务的其他单元必须等待该服务启动成功之后，才能启动。 但是对于 Type=simple 类型， 依赖于该服务的其他单元无需等待， 将会和该服务同时并行启动。 下面的类似展示了一个简单的静态防火墙服务：

[Unit]
Description=简单的静态防火墙

[Service]
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/local/sbin/simple-firewall-start
ExecStop=/usr/local/sbin/simple-firewall-stop

[Install]
WantedBy=multi-user.target
因为服务启动成功后一直处于"活动"(active)状态， 所以再次执行 systemctl start 命令不会有任何效果。


例 4. 传统的服务

多数传统的守护进程(服务)在启动时会转入后台运行。 systemd 通过 Type=forking 来支持这种工作方式。 对于这种类型的服务，如果最初启动的进程尚未退出， 那么该单元将依然处于"启动中"(activating)状态。 当最初的进程成功退出， 并且至少有一个进程仍然在运行(并且 RemainAfterExit=no)， 该服务才会被视为处于"活动"(active)状态。

对于单进程的传统服务，当最初的进程成功退出后， 将会只剩单独一个进程仍然在持续运行， systemd 将会把这个唯一剩余的进程视为该服务的主进程。 仅在这种情况下，才将可以在 ExecReload=, ExecStop= … 之类的选项中使用 $MAINPID 变量。

对于多进程的传统服务，当最初的进程成功退出后，将会剩余多个进程在持续运行， 因此，systemd 无法确定哪一个进程才是该服务的主进程。 在这种情况下，不可以使用 $MAINPID 变量。 然而，如果主进程会创建传统的PID文件， 那么应该将 PIDFile= 设为此PID文件的绝对路径， 以帮助 systemd 从该PID文件中读取主进程的PID，从而帮助确定该服务的主进程。 注意，守护进程必须在完成初始化之前写入PID文件， 否则可能会导致 systemd 读取失败 (读取时文件不存在)。

下面是一个 单进程传统服务的示例：

[Unit]
Description=一个单进程传统服务

[Service]
Type=forking
ExecStart=/usr/sbin/my-simple-daemon -d

[Install]
WantedBy=multi-user.target
参见 systemd.kill(5) 以了解如何 结束服务进程。


例 5. DBus 服务

对于需要在 D-Bus 系统总线上注册一个名字的服务， 应该使用 Type=dbus 并且设置相应的 BusName= 值。 该服务不可以派生任何子进程。 一旦从 D-Bus 系统总线成功获取所需的名字，该服务即被视为初始化成功。 下面是一个典型的 D-Bus 服务：

[Unit]
Description=一个简单的 DBus 服务

[Service]
Type=dbus
BusName=org.example.simple-dbus-service
ExecStart=/usr/sbin/simple-dbus-service

[Install]
WantedBy=multi-user.target
对于基于 D-Bus 启动的服务 来说， 不可以包含 "[Install]" 小节， 而是应该在对应的 D-Bus service 文件中设置 SystemdService= 选项，例如 (/usr/share/dbus-1/system-services/org.example.simple-dbus-service.service):

[D-BUS Service]
Name=org.example.simple-dbus-service
Exec=/usr/sbin/simple-dbus-service
User=root
SystemdService=simple-dbus-service.service
参见 systemd.kill(5) 手册以了解如何 结束服务进程。


例 6. 能够通知初始化已完成的服务

Type=simple 类型的服务 非常容易编写， 但是， 无法向 systemd 及时通知 "启动成功"的消息， 是一个重大缺陷。 Type=notify 可以弥补该缺陷， 它支持将"启动成功"的消息及时通知给 systemd 。 下面是一个典型的例子：

[Unit]
Description=Simple notifying service

[Service]
Type=notify
ExecStart=/usr/sbin/simple-notifying-service

[Install]
WantedBy=multi-user.target
注意， 该守护进程必须支持 systemd 通知协议， 否则 systemd 将会认为该服务一直处于"启动中"(activating)状态，并在超时后将其杀死。 关于如何支持该通知协议，参见 sd_notify(3) 手册。

参见 systemd.kill(5) 手册以了解如何 结束服务进程。
