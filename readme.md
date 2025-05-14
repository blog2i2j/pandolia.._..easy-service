中文 | [English](./readme.eng.md)

### 介绍

如果你的 Windows 程序需要在后台长期运行，而且你希望它在开机后用户登录之前就自动运行、且在用户注销之后也不停止，那么你需要将程序注册为一个系统服务。

然而，在 Windows 下编写一个可注册为系统服务的程序并不是一件简单的事情。首先，程序必须是二进制的可执行程序，这就排除了脚本语言和虚拟机语言；其次，程序必须按系统服务的格式编写，过程繁琐，编写示例可见：[MS 官方文档](https://code.msdn.microsoft.com/windowsapps/CppWindowsService-cacf4948) 。

[EasyService](https://github.com/pandolia/easy-service) 是一个可以将常规程序注册为系统服务的工具，体积只有 37KB 。你可以按常规的方法编写程序，然后用 EasyService 注册为一个系统服务，这样你的程序就可以在开机后用户登录之前自动运行、且在用户注销之后也不会停止。

如果你需要在 Windows 系统下部署网站、API 或其他需要长期在后台运行的服务， EasyService 将是一个很有用的工具。

### 安装

下载 [源码及程序](https://github.com/pandolia/easy-service/archive/master.zip)，解压。右键单击 bin 目录下的 register-this-path.bat ，以管理员身份运行，将 bin 目录加入至系统路径中，也可以手动将此目录加入至系统路径。

重新打开 “我的电脑” ，在任意位置打开一个命令行窗口，输入 ***svc -v*** ，如果正常输出版本信息，则表明安装成功。

### 使用方法

（1） 编写、测试你的程序，EasyService 对程序仅有一个强制要求和一个建议：

* 强制要求： 程序应持续运行

* 建议： 当程序的标准输入接收到 “exit” 或 “回车” 后在 5 秒之内退出

其中建议要求是非强制性的，程序不满足此要求也可以。

典型的程序见 [index.js](https://github.com/pandolia/easy-service/blob/master/samples/nodejs-version/worker/index.js) （nodejs 版）， [main.py](https://github.com/pandolia/easy-service/blob/master/samples/python-version/worker/main.py) （python 版） 或 [SampleWorker.cs](https://github.com/pandolia/easy-service/blob/master/src/SampleWorker.cs) （C# 版），这三个程序都是每隔 1 秒打印一行信息，键入回车后退出。

（2） 打开命令行窗口，输入 ***svc create hello-svc*** ，将创建一个样板工程 hello-svc 。

（3） 打开 hello-svc/svc.conf 文件，修改配置：

```conf
# Windows 系统服务名称、不能与系统中已有服务重名
ServiceName: hello-svc

# 需要运行的可执行程序及命令行参数
Worker: sample-worker.exe

# 程序运行的工作目录，请确保该目录已存在
WorkingDir: worker

# 输出目录，程序运行过程的输出将会写到这个目录下面，请确保该目录已存在
# 如果不想保存 Worker 的输出，请设为 $NULL
OutFileDir: outfiles

# 程序输出的编码，如果不确定，请设为空
WorkerEncoding: utf8
```

（4） 用管理员身份打开命令行窗口（Win10 系统下，需要在开始菜单中搜索 cmd 然后右键以管理员身份运行）， cd 到 hello-svc 目录：

a. 运行 ***svc check*** 命令检查配置是否合法

b. 运行 ***svc test-worker*** 命令测试 Worker 程序是否能正常运行

若测试无误：

c. 运行 ***svc install*** 命令安装并启动系统服务，此时程序就已经开始在后台运行了

d. 运行 ***svc stop|start|restart|remove*** 停止、启动、重启或删除本系统服务

### 注册多个服务

如果需要注册多个服务，可以用 ***svc create*** 创建多个目录，修改 svc.conf 中的服务名和程序名等内容，再在这些目录下打开命令行窗口执行 svc check|test-worker|install 等命令就可以了。需要注意的是：

* （1） 不同目录下的服务名不能相同，也不能和系统已有的服务同名

* （2） 配置文件中的 Worker/WorkingDir/OutFileDir 都是相对于该配置文件的路径

* （3） 注册服务之前，WorkingDir/OutFileDir 所指定的目录必须先创建好

### EasyService 服务操作命令

以下命令可以操作 EasyService 服务（用 svc 命令注册的服务），这些命令可在任意位置运行，不需要 cd 到 svc.conf 所在的目录：

* svc list|ls： 列出所有服务

* svc start|stop|remove all： 启动、停止或删除所有服务

* svc check|status|test-worker|install|start|stop|restart|remove $project-directory： 操作 $project-directory 目录下 svc.conf 指定的服务（$project-directory 中必须含有字符 \\ 或 /）

* svc start|stop|restart|remove $service-index： 操作第 $service-index 个服务（$service-index 为数字，运行 svc ls 可查看所有服务的序号）

* svc start|stop|restart|remove $service-name： 操作名称为 $service-name 的服务（$service-name 不全为数字、不包含 \\ 或 / ，且不为 all ）

以 start 命令为例： 

* svc start all： 启动所有服务

* svc start aa\： 启动 aa\svc.conf 文件指定的服务

* svc start aa： 启动名称为 aa 的服务

* svc start 2： 启动第 2 个服务

注意： check|status|test-worker|install 命令只支持 $project-directory 模式， restart 命令不支持 all 模式。

### 注意事项

为保证服务的一致性，要求：

* （1） 运行 ***svc install*** 安装服务之后，在运行 ***svc remove*** 删除服务之前，不应对 svc.conf 文件进行修改，删除，也不应移动或重命名目录

* （2） 不应在服务管理控制台中对采用 EasyService 安装的服务进行修改或操作，也不应采用除 svc 命令以外的其他方式进行修改或操作

### 配置项

```conf
# 服务名称，不能与系统中已有服务重名
ServiceName: easy-service

# 服务显示名称，不能与系统中已有服务的显示名称相同
DisplayName: easy-service

# 服务描述
Description: An example of EasyService

# 本服务依赖的服务名列表，用逗号分开，例如： Appinfo,AppMgmt
Dependencies:

# 需要运行的可执行程序及命令行参数
Worker: sample-worker.exe

# 运行程序的环境变量
Environments: TEST-ENV1=A1,TEST-ENV2=A2,TEST-ENV3=A3
Environments: TEST-ENV4=A4,TEST-ENV5=A5,TEST-ENV6=A6

# 程序运行的工作目录，请确保该目录已存在
WorkingDir: worker

# 输出目录，程序运行过程的输出将会写到这个目录下面，请确保该目录已存在
# 如果不想保存 Worker 的输出，请设为 $NULL
OutFileDir: outfiles

# 停止服务时，等待程序主动退出的最大时间
WaitSecondsForWorkerToExit: 5

# 注意： MaxLogFilesNum配置已废弃（自v1.0.11）
# 日志文件的最大个数，设为空则不限制，否则需要设置为大于等于 2 的整数
# svc.exe 每隔两个小时检查一次日志文件个数，如果个数大于这个值，就删除最老的几个文件
MaxLogFilesNum: 

# 程序的内存使用限制值
WorkerMemoryLimit:

# 程序输出的编码，如果不确定，请设为空
WorkerEncoding: utf8

# 运行本服务的用户，一般情况下用 LOCAL-SYSTEM ，下面三个项都设为空
# 如果使用普通用户，下面三个项， 域、用户名、密码都要设置，而且要在服务管理面板里面授权此用户运行服务的权限
# 域如果不确定，可以设置为 “."
Domain:
User:
Password:
```

### 内部实现

EasyService 实质是将自己（svc.exe）注册为一个系统服务，此服务启动时，会读取 svc.conf 中的配置，创建一个子进程运行 Worker 中指定的程序及命令行参数，之后，监视该子进程，如果发现 ***子进程停止运行（或内存使用量超过指定值）*** ，会重新启动一个子进程。而当此服务停止时，按以下原则终止子进程：

* 当 WaitSecondsForWorkerToExit 为 0 时，直接终止子进程

* 当 WaitSecondsForWorkerToExit 不为 0 时，向子进程的标准输入中写入数据 “exit” ，并等待子进程退出，若等待时间超过 WaitSecondsForWorkerToExit 秒，则终止子进程。

EasyService 源码见 [src](https://github.com/pandolia/easy-service/tree/master/src) 。

### 与 NSSM 的对比

Windows 下部署服务的同类型的工具还有 NSSM ，与 EasyService 相比， NSSM 主要优点有：

* 提供了图形化安装、管理服务的界面

NSSM 主要缺点是界面和文档都是英文的，对新手也不见得更友好，另外在远程通过命令行编辑和管理服务稍微麻烦一些，需要记住它的命令的参数。

总体而言， EasyService 已实现了大部分服务程序需要的功能，主要优点有：

* 在命令行模式下编辑、管理和查看服务更方便

* 日志自动按日期输出到不同文件

* 停止服务时，可先向工作进程的标准输入写入 "exit" ，并等待工作进程自己退出，这个 “通知退出” 的机制对于需要进行清理工作的程序来说是非常关键的

### 典型用例

Appin 网站介绍了用 EasyService 部署 frp 内网穿透服务的方法，请看 [这里](https://www.appinn.com/easyservice-for-windows/) 。

### 意见反馈

若对 EasyService 有任何意见或建议，可以提 ISSUE ，也可以加作者微信 Kotlinfinity 。
