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
### 2.2.  /init
~~~
### 2.4. /sh (the shell)
Now we’re at sh.c (the source for /sh).
This is the program you interact with — the prompt you see in QEMU.
It waits for you to type commands.
For each command:
- Calls fork().
- Child calls exec(command, argv).
- Parent waits.
So every command you type spawns a new process.
### (2.5. hands over to scheduler)
