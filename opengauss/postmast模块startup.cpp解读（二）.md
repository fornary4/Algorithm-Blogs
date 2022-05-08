## postmaster模块startup.cpp解读（二）

**startup.cpp重要函数解读**



startupproc_quickdie的作用是快速杀死启动进程，在postmaster发出退出信号时执行

```c++
static void startupproc_quickdie(SIGNAL_ARGS)
{
    gs_signal_setmask(&t_thrd.libpq_cxt.BlockSig, NULL);

    /*
     * 我们不想运行 proc_exit() 回调——我们在这里是因为共享内存可能已损坏，
     * 所以我们不想尝试清理我们的事务。
     */
    on_exit_reset();

    /*
     * 这里执行 exit(2) 而不是 exit(0)。
     */
    exit(2);
}
```



StartupProcShutdownHandler的作用是设置中止重做和退出标志

```c++
static void StartupProcShutdownHandler(SIGNAL_ARGS)
{
    int save_errno = errno;

    if (t_thrd.startup_cxt.in_restore_command)
        proc_exit(1);
    else
        t_thrd.startup_cxt.shutdown_requested = true;

    WakeupRecovery();

    errno = save_errno;
}
```



HandleStartupProcInterrupts的作用是处理启动过程SIGHUP和SIGTERM信号

```c++
void HandleStartupProcInterrupts(void)
{
    /*
     * 检查是否被要求重新读取配置文件。
     */
    if (t_thrd.startup_cxt.got_SIGHUP) {
        t_thrd.startup_cxt.got_SIGHUP = false;
        ProcessConfigFile(PGC_SIGHUP);
    }

    /*
     * 检查是否被要求退出而没有完成恢复。
     */
    if (t_thrd.startup_cxt.shutdown_requested && SmartShutdown != g_instance.status) {
        proc_exit(1);
    }

    /*
     * postmaster进程结束后的紧急处理，这是为了避免手动清理所有 postmaster 子项的必要性。
     */
    if (IsUnderPostmaster && !PostmasterIsAlive())
        gs_thread_exit(1);
}
```



StartupReleaseAllLocks的作用是释放所有系统锁

```c++
static void StartupReleaseAllLocks(int code, Datum arg)
{
    Assert(t_thrd.proc != NULL);

    /* 如果不处于就绪模式就什么都不做 */
    if (t_thrd.xlog_cxt.standbyState == STANDBY_DISABLED)
        return;

    /* 如果在等待，退出等待队列(应该只在发生错误后才需要) */
    LockErrorCleanup();
    /* 释放标准锁，如果中止，则释放会话级锁 */
    LockReleaseAll(DEFAULT_LOCKMETHOD, true);

    /*
     * 用户锁不会在事务结束时释放，所以一定要释放
     * 明确
     */
    LockReleaseAll(USER_LOCKMETHOD, true);
}
```



PreRestoreCommand的作用是提前存储控制命令

```c++
void PreRestoreCommand(void)
{
    /*
     * 设置 t_thrd.startup_cxt信号来告诉信号处理程序我们应该在 SIGTERM 上立即退出。我们知道我们现在可以安全地做到这一点。检      * 查我们是否已经收到信号，这样我们就不会错过在此之前收到的关闭请求。
     */
    t_thrd.startup_cxt.in_restore_command = true;

    if (t_thrd.startup_cxt.shutdown_requested) {
        proc_exit(1);
    }
}
```



CheckNotifySignal的作用是检查notify sinal共享内存，找出信号产生的原因。

```c++
bool CheckNotifySignal(NotifyReason reason)
{
    /* 这里要小心 --- 如果我们没有看到它设置，不要清除标志 */
    if (t_thrd.startup_cxt.NotifySigState->NotifySignalFlags[reason]) {
        t_thrd.startup_cxt.NotifySigState->NotifySignalFlags[reason] = false;
        return true;
    }
    return false;
}
```

