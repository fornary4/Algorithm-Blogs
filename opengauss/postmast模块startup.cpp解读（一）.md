## postmast模块startup.cpp解读（一）



**概述：startup.cpp会初始化服务器并执行任何已指定的恢复操作**



**backend启动的函数调用栈：**

```c++
PostmasterMain()
　  |->ServerLoop()
　      |->initMasks()
　      |->for(;;)
　          |->select()         <--监听端口
　          |->ConnCreate()     <--创建connection相关的数据结构
　          |->BackendStartup() <--建立后端进程backend process
　              |->PostmasterRandom()
　              |->canAcceptConnections()
　              |->fork_process()
　              |->InitPostmasterChild()
　              |->ClosePostmasterPorts()
　              |->BackendInitialize()
　                  |->ProcessStartupPacket()
　              |->BackendRun()
　                  |->PostgresMain()
            |->ConnFree()       <--释放connection相关的数据结构
```

在系统调用select()中，我们监听客户端的连接请求，当读到一个客户端请求时我们将为其创建相关数据结构，做一下初始化。注意此时只是监听接受了请求，这个请求是否合法(例如password是否正确)在此时是不做判断的，判断是放在BackendStartup()中的。

**client连接请求认证的函数调用栈：**

```c++

BackendRun()
    |->PostgresMain()
        |->InitPostgres()
            |->PerformAuthentication()
                |->ClientAuthentication()
```



**后端处理cancel的流程图**

![image-20210829164600507](C:\Users\DELL\AppData\Roaming\Typora\typora-user-images\image-20210829164600507.png)

**信号处理函数的定义**

```c++

static void startupproc_quickdie(SIGNAL_ARGS);
static void StartupProcSigUsr1Handler(SIGNAL_ARGS);
static void StartupProcSigHupHandler(SIGNAL_ARGS);
static void StartupProcSigusr2Handler(SIGNAL_ARGS);
static void SetStaticConnNum(void);
```

**信号处理的实现：**

startupproc_quickdie的作用是让进程快速结束，SigUsr1是调用锁去处理信号，Sigusr2是设置标志以完成恢复，SIGUSR2 启动过程的处理程序 当收到 SIGUSR2 时，检查 SIGUSR2 的原因并做相应的操作,StartupProcSigusr2Handler是设置标志以在下次方便的时候重新读取配置文件。

