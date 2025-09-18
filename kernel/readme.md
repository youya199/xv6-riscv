# The kernel booting sequence
## 1. Kernel starts up 
Boot ROM → entry.S → start.c → main.c → subsystem init → userinit() → [scheduler()](https://github.com/youya199/xv6-riscv/blob/riscv/kernel/proc.c#L422)

Inside main.c, after initializing subsystems, xv6 calls userinit().
## 2. userinit() (in [proc.c](https://github.com/youya199/xv6-riscv/blob/riscv/kernel/proc.c#L220))
This function creates the very first user process:
- Allocates a process slot in proc[].(the global process list)
- Sets up a trapframe (registers for entering user mode).
- Loads a tiny program initcode.S into its memory.
- Marks it as RUNNABLE.
## 3. initcode.S (the tiny bootstrap user program)
This is not the shell yet!
initcode.S is only a few instructions long.
Its job:
- Call the exec() system call with arguments "init".
- This tells the kernel: “replace me with the real /init program.”
So the very first process doesn’t run shell — it runs initcode, which immediately transforms into /init.
## 4. /init (the real first user program)
The file init.c (compiled into the xv6 file system as /init) is the first true user program.
It does some setup:
- Opens the console device.
- Duplicates file descriptors (so stdin/out/err point to console).
Then enters a loop:
~~~
for(;;){
    if(fork() == 0){
        exec("sh", argv);  // child runs /sh
    }
    wait(0);  // parent waits until child (the shell) exits
}
~~~
## 5. /sh (the shell)
Now we’re at sh.c (the source for /sh).
This is the program you interact with — the prompt you see in QEMU.
It waits for you to type commands.
For each command:
- Calls fork().
- Child calls exec(command, argv).
- Parent waits.
So every command you type spawns a new process.
## (6. hands over to scheduler)
