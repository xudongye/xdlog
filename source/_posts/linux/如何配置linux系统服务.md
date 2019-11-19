---
title: 如何配置linux系统服务?
date: 2019-11-18 17:10:38
categories: 
- Linux
tags: 
- Linux
- Centos
---

### Linux的系统服务daemon(守护进程；后台程序)

一、service与daemon
* service是从功能需求角度而言，daemon是从功能实现角度而言
* 服务依赖于daemon
* 当实现某种功能的进程Process常驻在内存，就成为了后台进程
* 常驻内存中Process是为了给系统提供稳定连续的基础服务Service,也就是后台进程提供了系统服务
* 系统需要各种服务，从而就需要各种进程

二、主机服务怎么与客户端联系
* linux主机 -> 运行着各种Process，以实现各种功能，某些Process因为常驻内存，故又称之为Daemon
* 客户端 -> 根据IP 联系到linux主机 -> 和主机哪个Process沟通？？？
* linux主机 -> 运行着各种Process / Daemon -> 每个 Process / Daemon 打开着相应的 Port 
* 客户端 -> 根据IP 和 Port 联系到主机具体的 Process / Daemon ！！！

三、Linux中系统服务的管理
* systemd开启和监督整个系统是基于unit的概念
* 如何理解unit单元
    * 系统服务需要各种各样的配置文件。unit是一组配置文件的封装单元，其命名规则是："服务名"+"."+"类型名"。unit 文件统一了过去各种不同的系统资源配置格式，例如服务的启/停、定时任务、设备自动挂载、网络配置、设备配置、虚拟内存配置等。而 Systemd 通过不同的后缀名来区分这些配置文件。所以，unit可以理解为两层意思：【单元】它是一个可以概括各种不同类型配置文件的单元体，我们可以称各种服务的配置文件为unit文件。【统一的】unit文件内容具有统一的格式。unit 文件内容的可以分为3个配置区段：[unit][service][install]。其中 Unit 和 Install 段是所有类型 Unit 文件通用的，用于配置服务（或其他系统资源）的描述、依赖和随系统启动方式。而 Service 段则是服务类型的 Unit 文件（后缀.service）特有的，用于定义服务的具体管理和操作方法。其他的每种配置文件也都会有一个特有的配置段，这就是几种不同 Unit 配置文件最明显的区别
* 来看看每个配置段常用的参数有哪些
    * 【Unit 段】
        * Description 一段描述这个 Unit 文件的文字，通常只是简短的一句话
        * Documentation 指定服务的文档，可以是一个或多个文档的URL路径
        * Requires 依赖的其他 Unit 列表，列在其中的 Unit 模块会在这个服务启动的同时被启动，并且如果其中有任意一个服务启动失败，这个服务也会被终止
        * Wants 与 Requires 相似，但只是在被配置的这个 Unit 启动时，触发启动列出的每个 Unit 模块，而不去考虑这些模块启动是否成功
        * After 与 Requires 相似，但会在后面列出的所有模块全部启动完成以后，才会启动当前的服务
        * Before 与 After 相反，在启动指定的任一个模块之前，都会首先确保当前服务已经运行
        * BindsTo 与 Requires 相似，但是一种更强的关联。启动这个服务时会同时启动列出的所有模块，当有模块启动失败时终止当前服务。反之，只要列出的模块全部启动以后，也会自动启动当前服务。并且这些模块中有任意一个出现意外结束或重启，这个服务会跟着终止或重启
        * PartOf 这是一个 BindTo 作用的子集，仅在列出的任何模块失败或重启时，终止或重启当前服务，而不会随列出模块的启动而启动
        * OnFailure 当这个模块启动失败时，就自动启动列出的每个模块
        * Conflicts 与这个模块有冲突的模块，如果列出模块中有已经在运行的，这个服务就不能启动，反之亦然
        * 上面这些配置中，除了 Description 外，都能够被添加多次。比如前面第一个例子中的After参数在一行中使用空格分隔指定所有值，也可以像第二个例子中那样使用多个After参数，在每行参数中指定一个值
    * 【Install 段】这个段中的配置与 Unit 有几分相似，但是这部分配置需要通过 systemctl enable 命令来激活，并且可以通过 systemctl disable 命令禁用。另外这部分配置的目标模块通常是特定启动级别的 .target 文件，用来使得服务在系统启动时自动运行
        * WantedBy 和前面的 Wants 作用相似，只是后面列出的不是服务所依赖的模块，而是依赖当前服务的模块
        * RequiredBy 和前面的 Requires 作用相似，同样后面列出的不是服务所依赖的模块，而是依赖当前服务的模块
        * Also 当这个服务被 enable/disable 时，将自动 enable/disable 后面列出的每个模块
        * “WantedBy=multi-user.target” 表明当系统以多用户方式（默认的运行级别）启动时，这个服务需要被自动运行。当然还需要 systemctl enable 激活这个服务以后自动运行才会生效。关于 Linux 系统启动时的运行级别，可以参看[这篇文章](https://www.linuxidc.com/Linux/2014-11/109239.htm)
    * 【Service 段】 这个段是 .service 文件独有的，也是对于服务配置最重要的部分。这部分的配置选项非常多，主要分为服务生命周期控制和服务上下文配置两个方面，下面是比较常用到的一些参数
        * 服务生命周期控制相关的参数
            * Type 服务的类型，常用的有 simple（默认类型） 和 forking。默认的 simple 类型可以适应于绝大多数的场景，因此一般可以忽略这个参数的配置。而如果服务程序启动后会通过 fork 系统调用创建子进程，然后关闭应用程序本身进程的情况，则应该将 Type 的值设置为 forking，否则 systemd 将不会跟踪子进程的行为，而认为服务已经退出
            * RemainAfterExit 值为 true 或 false（也可以写 yes 或 no），默认为 false。当配置值为 true 时，systemd 只会负责启动服务进程，之后即便服务进程退出了，systemd 仍然会认为这个服务是在运行中的。这个配置主要是提供给一些并非常驻内存，而是启动注册后立即退出然后等待消息按需启动的特殊类型服务使用 
            * ExecStart 这个参数是几乎每个 .service 文件都会有的，指定服务启动的主要命令，在每个配置文件中只能使用一次
            * ExecStartPre 指定在启动执行 ExecStart 的命令前的准备工作，可以有多个，所有命令会按照文件中书写的顺序依次被执行
            * ExecStartPost 指定在启动执行 ExecStart 的命令后的收尾工作，也可以有多个
            * TimeoutStartSec 启动服务时的等待的秒数，如果超过这个时间服务任然没有执行完所有的启动命令，则 systemd 会认为服务自动失败。这一配置对于使用 Docker 容器托管的应用十分重要，由于 Docker 第一次运行时可以能会需要从网络下载服务的镜像文件，因此造成比较严重的延时，容易被 systemd 误判为启动失败而杀死。通常对于这种服务，需要将 TimeoutStartSec 的值指定为 0，从而关闭超时检测
            * ExecStop 停止服务所需要执行的主要命令
            * ExecStopPost 指定在 ExecStop 命令执行后的收尾工作，也可以有多个
            * TimeoutStopSec 停止服务时的等待的秒数，如果超过这个时间服务仍然没有停止，systemd 会使用 SIGKILL 信号强行杀死服务的进程
            * Restart 这个值用于指定在什么情况下需要重启服务进程。常用的值有 no，on-success，on-failure，on-abnormal，on-abort 和 always。默认值为 no，即不会自动重启服务。这些不同的值分别表示了在哪些情况下，服务会被重新启动
            * RestartSec 如果服务需要被重启，这个参数的值为服务被重启前的等待秒数
            * ExecReload 重新加载服务所需执行的主要命令
        * 服务上下文配置相关的参数
            * Environment 为服务添加环境变量
            * EnvironmentFile 指定加载一个包含服务所需的环境变量列表的文件，文件中的每一行都是一个环境变量的定义
            * Nice 服务的进程优先级，值越小优先级越高，默认为0。-20为最高优先级，19为最低优先级
            * WorkingDirectory 指定服务的工作目录
            * RootDirectory 指定服务进程的根目录（ / 目录），如果配置了这个参数后，服务将无法访问指定目录以外的任何文件
            * User 指定运行服务的用户，会影响服务对本地文件系统的访问权限
            * Group 指定运行服务的用户组，会影响服务对本地文件系统的访问权限
            * LimitCPU / LimitSTACK / LimitNOFILE / LimitNPROC 限制特定服务可用的系统资源量，例如 CPU，程序堆栈，文件句柄数量，子进程数量… 不再展开说明了，值的含义可参考 [Linux 文档](http://man7.org/linux/man-pages/man2/setrlimit.2.html#DESCRIPTION)资源配额部分中 RLIMIT_ 开头的那些参数
* unit文件位置 
    * Unit 文件按照 Systemd 约定，应该被放置在指定的3个系统目录之一。这3个目录是有优先级的，依照下面表格，越靠上的优先级越高，因此在几个目录中有同名文件的时候，只有优先级最高的目录里的那个会被使用
---
路径 | 说明 
---|---
/etc/systemd/system | 系统或用户提供的配置文件
/run/systemd/system | 软件运行时生成的配置文件
/usr/lib/systemd/system | 系统或第三方软件安装时添加的配置文件  
---   
* unit模板文件 
    * 在现实中，往往有一些应用需要被复制多份运行，例如在一个负载均衡实例后面运行的多个相同的服务实例。但是按照之前的例子，每个服务都需要一个单独的 Unit 文件，这样复制多份相同文件的做显然不便于服务的管理。为此 Systemd 定义了一种特殊的 Service Unit文件，称为 Unit 模板
    * 模板文件的主要特点是，文件名以@符号结尾，而启动的时候指定的Unit名称为模板名称附加一个参数字符串。例如，将之前的例子第二个 Unit 文件修改为可以用于启动多个实例的模板
    * 首先修改文件名，添加一个@符号。
      例如原来的文件名是 apache.service，那么可以将它修改为 apache@.service，这样做的目的是表面这个文件是一个模板文件。而在服务启动时可以在@后面放置一个用于区分服务实例的附加字符串参数，通常这个参数会使用监听的端口号或使用的控制台TTY编号等。例如 “systemctl start apache@8080.service”
    * 然后修改 Unit 文件内容。
      Unit 文件中可以获取服务启动时的附加参数，因此通常需要修改 Unit 文件中不应固定的部分，例如服务监听的 IP 和端口，替换为从附加参数中获取
  ````
  [Unit]
  Description=My Advanced Service Template
  After=etcd.servicedocker.service
  [Service]
  TimeoutStartSec=0
  ExecStartPre=-/usr/bin/docker kill apache%i
  ExecStartPre=-/usr/bin/docker rm apache%i
  ExecStartPre=/usr/bin/docker pull coreos/apache
  ExecStart=/usr/bin/docker run --name apache%i -p %i:80 coreos/apache /usr/sbin/apache2ctl -D FOREGROUND
  ExecStartPost=/usr/bin/etcdctl set /domains/example.com/%H:%i running
  ExecStop=/usr/bin/docker stop apache1
  ExecStopPost=/usr/bin/etcdctl rm /domains/example.com/%H:%i
  [Install]
  WantedBy=multi-user.target
  ````
    * 仔细观察一下变化了的地方，上面使用到了占位符 %H 和 %i，常用的占位符有6种（一共19种，其余不怎么常用的查文档吧），这些占位符会在 Unit 启动时被实际的值动态的替换掉
    * 这些参数中除了 %i 以外，同样可以用于非模板的 Unit 文件中。%p 在普通 Unit 文件中会被动态替换为服务名称去掉 .service 后缀的名字
---
占位符 | 作用
---|---
%n | 完整的 Unit 文件名字，包括 .service 后缀名
%m | 实际运行的节点的 Machine ID，适合用来做Etcd路径的一部分，例如 /machines/%m/units
%b | 作用有点像 Machine ID，但这个值每次节点重启都会改变，称为 Boot ID
%H | 实际运行节点的主机名
%p | Unit 文件名中在 @ 符号之前的部分，不包括 @ 符号
%i | Unit 文件名中在 @ 符号之后的部分，不包括 @ 符号和 .service 后缀名
---

* 启动 Unit 模板的服务实例
    * 模板服务的启动对于 Systemd 和 Fleet 大致相同
    * Systemd 的情况略简单一+些，只需要运行时加上后缀参数。例如 “systemctl start apache@8080.service”。Systemd 首先会在其特定的目录下寻找名为 apache@8080.service的文件，如果没有找到，而文件名中包含@字符，它就会尝试去掉后缀参数匹配模板文件。例如没有找到apache@8080.service，那么Systemd会找到apache@.service，并将它通过模板文件中实例化
    * Fleet 没有特定的 Unit 文件存放目录，不过在通过 fleetctl start 或 fleetctl submit 命令指定 Unit 文件路径时加上后缀参数，Fleet 同样会自动匹配去掉后缀参数后的模板文件。例如 “fleetctl submit ${HOME}/apache@8080.service”，就会匹配到 ${HOME} 目录下面的 apache@.service 模板文件
* systemctl命令 系统服务的控制-启动、关闭、重启；注销、取消注销
    * 启动服务 
        * 格式：systemctl start [unit type] 
        * systemctl start network.service
        * 对比之前的 service命令
        * 格式：service [服务] start
        * service network start
    * 停止服务
        * 格式：systemctl stop [unit type] 
    * 重启服务
        * 格式：systemctl restart [unit type]
    * 重新加载服务
        * 格式：systemctl reload [unit type] 
        * 其功能是重新加载服务，加载更新后的配置
    * 注销指定服务
        * 格式：systemctl mask [unit type] 
    * 取消注销指定服务
        * 格式：systemctl unmask [unit type] 
    * 关闭网络服务
        * 在使用systemctl关闭网络服务时有一些特殊 需要同时关闭unit.servce和unit.socket。比如说，如果 sshd.service 和 sshd.socket 都开启了，如果只关闭了sshd.service , 那么 sshd.socket 还在监听网络，在网络上如果有要求连接ssh时就会启动 sshd.service。所以想完全关闭ssh的话，需.service和 .socket同时关闭
            * systemctl stop sshd.service
            * systemctl stop sshd.socket
            * systemctl disable sshd.service sshd.socket 
* 设置服务的开机启动/不启动——对应之前的chkconfig命令
    * 设置服务开机启动。
    * 格式：systemctl enable [unit type] 
    * 设置服务开机不启动
    * 格式：systemctl disable [unit type] 
    * 查看服务运行状态
    * 格式：systemctl status [unit type]
* 查看系统上的所有服务
    * 格式：systemctl -list-units [-all] [-type=TYPE]  依据unit列出所有启动的unit[-all 包括未启动的][指定TYPE的服务]
    * 格式：systemctl -list-unit-files [-all] [-type=TYPE]  依据/usr/lib/systemd/system/ 内的启动文件，列出启动文件列表
````
#列出所有的系统服务
systemctl    

#列出所有启动unit
systemctl list-units    

#列出所有启动文件
systemctl list-unit-files    

#列出所有service类型的unit
systemctl list-units –type=service –all    

#列出 cpu电源管理机制的服务
systemctl list-units –type=service –all grep cpu    

#列出所有target
systemctl list-units –type=target –all
````
* 查看服务是否运行
    * 格式：systemctl is-active [unit type]
* 查看服务是否为开机启动
    * 格式：systemctl is-enable [unit type]
* 控制开关机
    * 系统关机 systemctl poweroff
    * 对应之前init命令 init 0
* 重新启动
    * systemctl reboot
    * init 6
* 进入睡眠模式
    * systemctl suspend
* 进入休眠模式
    * systemctl hibernate
* 强制进入救援模式
    * systemctl rescue
* 强制进入紧急救援模式
    * systemctl emergency
* 设置系统运行级别
    * 取得当前的target 
    * systemctl get-default 
    * 设置指定的target为默认的运行级别
    * 格式：systemctl set-default [unit.target]
````
#设置默认的运行级别为multi-user
systemctl set-default multi-user.target
````
* 切换到指定的运行级别
    * 格式：systemctl isolate [unit.target]
````
#在不重启的情况下，切换到运行级别multi-user下
systemctl isolate multi-user.target
#在不重启的情况下，切换到图形界面下
systemctl isolate graphical.target
````

* 分析各服务之间的依赖关系
    * 格式：systemctl list-dependencies [unit.type] [-reverse]   此unit依赖谁，[--reverse]则反之
````
#查看当前运行级别target启动了哪些服务。
systemctl list-dependencies

#查看哪些服务引用了当前运行级别target。 
systemctl list-dependencies --reverse
````
    