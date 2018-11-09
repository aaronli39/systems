# SYSTEMS

## 11/08/18 - signals

- you can man 7 signals to read more about signals
- seg faults are sent by OS to terminate code(kill - 11 gives seg  fault errors)
- kill -SIGSTOP PID stops a program, and SIGRES can resume it
- after killing a process, it may still be in the foreground, so you can type(fg) to get rid of them
- killall -9 a.out will immediately completely remove a process
- there has to be a static void function sighandler in C(it cant be called from other files):

```
static void sighandler(int signo) {
	printf("signo: ");
	if (signo == SIGSEGV) {
		printf("SEGGED\n");
	}
	if (signo == SIGUSR1 {
		printf("whadayya want from me!\n");
	}
}

int main () {
	signal(SIGSEGV, sighandler); // this allows sigsev to be handled

}

```

Signals in C programs: ```<signal.h>```

**kill**

- ```kill(pid, signal); ```
- returns 0 on success or -1 if not

**signal**

- ```signal(SIGNUM, sighandler) ```
- ```getpid();``` gets the pid of running process
- ```SIGKILL``` cannot be intercepted
- you dont want to ```kill -9``` your web server/bash etc because there are some things it must do before it closes, and it cant do so if you force it to close.
- ```kill -L``` lists all processes you can kill
* files under /proc lists all files such as uptime



## 11/07/18 - Processes
CTRL-D closes file

**Proccesses:**

- every running program is a process
- a process can create subprocesses, but these are no different from regular processses
- a processor can handle 1 process per cycle(per core)
- multitasking appears to happen because the processor switches between all the active
  ​    processes quickly
- on *nix style machines, processes are also files, listed in the /proc directory
- for eg: an 8-core processor can handle 8 processes in one cycle
- ps -a shows all processes
- ps -au shows processes and USERS
- ps -au | grep aaronli will show only aaronli's processes
- fd: file descriptors
- theres metadeta on every process thats running and thats how commands like ps work
- each process you run can have different environmental variables

**PID**

- every process has a unique identifier called the pid
- pid 1 is the init process
- getpid() -  ```<unistd.h>```
- returns the current process's pid
- getppid() - <unistd.h>
  - returns the current process's parent pid
  - What happens when you close a parent process?
    - subprocesses DO NOT ALWAYS close
    - an orphaned child's ppid will be set to 1 to show parent is not there
    - orphaned processes dont get "Adopted"
  - you can kill a process with its PID
  - $kill -9 bash: closes current bash and all children
  - $kill -9 -1: logs you out
* asdf
11/07/18 -- Processes

* CTRL-D closes file

Proccesses:
* every running program is a process
* a process can create subprocesses, but these are no different from regular processses
* a processor can handle 1 process per cycle(per core)
* multitasking appears to happen because the processor switches between all the active processes quickly
* linix style machines, processes are also files, listed in the /proc directory
* for eg: an 8-core processor can handle 8 processes in one cycle
* ps -a shows all processes
* ps -au shows processes and USERS?
* ps -au | grep aaronli will show only aaronli's processes
* fd: file descriptors
* theres metadeta on every process thats running and thats how commands like ps work
* each process you run can have different environmental variables

PID:
* every process has a unique identifier called the pid
* pid 1 is the init process
* getpid() - <unistd.h>
	* returns the current process's pid
    * getppid() - <unistd.h>
        * returns the current process's parent pid
* What happens when you close a parent process?
    * subprocesses **DO NOT ALWAYS** close
	* an orphaned childs ppid will be set to 1 to show parent is not there
        * orphaned processes dont get "Adopted"
    * you can kill a process with its PID
    * ```kill -9 bash```: closes current bash and all children
    * ```kill -9 -1```: logs you out
