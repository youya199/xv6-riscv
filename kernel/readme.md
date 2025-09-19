# The kernel booting sequence
## 1. Kernel starts up 
Boot ROM → entry.S → start.c → main.c → subsystem init → userinit() → [scheduler()](https://github.com/youya199/xv6-riscv/blob/riscv/kernel/proc.c#L422)

Inside main.c, after initializing subsystems, xv6 calls userinit().
## 2. INSIDE main.c
### 2.1. userinit() (in [proc.c](https://github.com/youya199/xv6-riscv/blob/riscv/kernel/proc.c#L220))
This function creates the very first user process:
- Allocates a process slot in proc[].(the global process list)
    - Sets up a trapframe (registers for entering user mode).
    - sets /init as the content binary of the first process
- Marks it as RUNNABLE.
### 2.2. where is /init from
It is in [/user/init.c](https://github.com/youya199/xv6-riscv/blob/riscv/user/init.c#L15). Here we did:

$exec("sh", argv);$
### 2.3. Go in scheduler()
Scheduler finds out the runnable process, which is /init, then it runs that, and it turns to shell with exec.

At last maybe shell waits for your input, this waiting gives control back to scheduler.
### 2.4. /sh (the shell)
Now we’re at sh.c (the source for /sh).
This is the program you interact with — the prompt you see in QEMU.
It waits for you to type commands.
For each command:
- Calls fork().
- Child calls exec(command, argv).
- Parent waits.
So every command you type spawns a new process.
### (2.4. hands over to scheduler)







# The syscalls
## 1. Overview
- Definition / dispatcher of the 21 syscalls lives in syscall.c.
~~~
static uint64 (*syscalls[])(void) = {
[SYS_fork]   sys_fork,
[SYS_exit]   sys_exit,
[SYS_wait]   sys_wait,
[SYS_pipe]   sys_pipe,
[SYS_read]   sys_read,
[SYS_kill]   sys_kill,
[SYS_exec]   sys_exec,
[SYS_fstat]  sys_fstat,
[SYS_chdir]  sys_chdir,
[SYS_dup]    sys_dup,
[SYS_getpid] sys_getpid,
[SYS_sbrk]   sys_sbrk,
[SYS_sleep]  sys_sleep,
[SYS_uptime] sys_uptime,
[SYS_open]   sys_open,
[SYS_write]  sys_write,
[SYS_mknod]  sys_mknod,
[SYS_unlink] sys_unlink,
[SYS_link]   sys_link,
[SYS_mkdir]  sys_mkdir,
[SYS_close]  sys_close,
};
~~~
- Implementations are spread across files like sysproc.c, sysfile.c, etc.
## 2. Where each syscall is implemented
1. kernel/sysproc.c **(process-related)**
- sys_fork → calls fork()
- sys_exit → calls exit()
- sys_wait → calls wait()
- sys_kill → calls kill()
- sys_getpid → returns current process’s PID
- sys_sbrk → grows process memory
- sys_sleep → sleep for N ticks
- sys_uptime → return system uptime in ticks
2. kernel/sysfile.c **(file system & I/O)**
- sys_pipe → create a pipe
- sys_read → read from fd
- sys_write → write to fd
- sys_close → close fd
- sys_dup → duplicate fd
- sys_fstat → get file status
- sys_chdir → change working dir
- sys_exec → replace process image
- sys_mknod → create device file
- sys_unlink → delete file/dir entry
- sys_link → create a hard link
- sys_mkdir → create directory
- sys_open → open file
3. User space wrappers
In user/user.h → user programs see these as normal function prototypes:
~~~
int fork(void);
int exit(int);
int wait(int*);
...
~~~
These are thin wrappers around the ecall instruction, implemented in user/usys.pl → user/usys.S. 

They just:
- Loads syscall number into a7
- Executes ecall
For example from usys.S:
~~~
.global fork
fork:
 li a7, SYS_fork
 ecall
 ret
~~~
