---
layout: post
title: "在JOS上实现基于Priority的RR_sched"
categories: jos
tags: [kernel]

---
在JOS上实现基于Priority的RR_sched
===================================
JOS在一开始实现的是简单的RR算法，没有优先级调度。

下面我实现了具有Priority的RR调度算法。首先我们需要增加一个sys_env_set_priority（）的系统调用。

首先在inc/env.h 的struct Env中增加env_priority字段。这个字段来表示进程的优先级。

然后再inc/env.h中增加__PRIORITY_HIGH PRIORITY_DEFAULT__ …..等宏变量。

在kern/env.h 的 env_alloc（）增加对env.env_priority的设定，默认为PRIORITY_DEAULT。

然后在真正的kern/syscall.c中实现该函数sys_env_set_priority（）。

下面我们对添加JOS系统中断

* 在inc/syscall.h中添加新的中断号：SYS_env_set_priority。

* 在kern/syscall.c的syscall()函数中添加相应switch分发语句。

* 在lib/syscall.c中添加供用户调用系统调用的库函数。

<pre><code>
int sys_env_set_priority(envid_t envid, int priority)
{
return syscall(SYS_env_set_priority, 1, envid, priority, 0, 0, 0);
}
</code></pre>

万事俱备，只欠东风。

下面在kern/sched.c中编写真正的调度程序 void sched_yield(void)
<pre><code>
void RR_Priority_sched(void)
{
       int now_env,i;
       if(curenv)
       {
           now_env = (ENVX(curenv->env_id) +1) % NENV;
       }
       else
       {
           now_env = 0 ;
       }

       uint32_t max_priority = 0;
       int select_env = -1;

   //  cprintf("NENV=%d\n",NENV);
       for(i= 0;i< NENV;i++ , now_env = (now_env+1)%NENV)      
       {          
            if(envs[now_env].env_status ==ENV_RUNNABLE && (envs[now_env].env_priority > max_priority
                       ||select_env == -1))
           {
               select_env=now_env;
               max_priority = envs[now_env].env_priority;
               cprintf ("I am CPU %d , I am in sched yield , I find ENV %d,Priority 0x%x, i = %d\n",
                       thiscpu ->cpu_id , select_env,max_priority,i);
           }
       }

       //cprintf ("I am CPU %d , I am in sched yield , I find ENV %d,Priority %d\n",thiscpu ->cpu_id , select_env,max_priority);

       if (select_env >= 0 && (! curenv || curenv ->env_status != ENV_RUNNING ||
                   max_priority >= curenv ->env_priority))
       {
           env_run (& envs[select_env ]);
       }
       if (curenv && curenv ->env_status == ENV_RUNNING)
       {
           env_run(curenv);
       }

   }
</code></pre>
系统调用在linux中差不多也是这个流程。[http://blog.csdn.net/m6830098/article/details/9056457]()

一般的，进程是不能访问内核的。它不能访问内核所占内存空间也不能调用内核函数。CPU硬件决定了这些（这就是为什么它被称作”保护模式”）。系统调用是这些规则的一个例外。其原理是进程先用适当的值填充寄存器，然后调用一个特殊的指令，这个指令会跳到一个事先定义的内核中的一个位置（当然，这个位置是用户进程可读但是不可写的）。在Intel CPU中，这个由中断0×80实现。硬件知道一旦你跳到这个位置，你就不是在限制模式下运行的用户，而是作为操作系统的内核–所以你就可以为所欲为。
进程可以跳转到的内核位置叫做sysem_call。这个过程检查系统调用号，这个号码告诉内核进程请求哪种服务。然后，它查看系统调用表(sys_call_table)找到所调用的内核函数入口地址。接着，就调用函数，等返回后，做一些系统检查，最后返回到进程（或到其他进程，如果这个进程时间用尽）。如果你希望读这段代码，它在<内核源码目录>/kernel/entry.S，Entry(system_call)的下一行。

传统意义上系统调用产生中断，陷入内核。

![](/assets/pic/IMG_0343.JPG)
