# SYSTEMS - Aaron Li

## 12/07/18 - shared memory

* pipes are nice but having a block of memory to write to is much nicer
* memory/stuff that remains even after a process has ended

#### shared memory ```<sys/shm.h>, <sys/ipc.h>, <sys/types.h>```

* if two processes try to access same memory at the same time then it will crash
* a segment of heap memory that can be accessed by mupltiple processes

#### Shared memory(cont)

**5 shared memory operations:**

#### ftok - ```<sys/ipc.h>```
* generate a key useful for IPC functions
* use this to check to make sure you create a UNIQUE block of memory
* ```ftok( path, x )```
	* path: a path to some file, the file msut be accessible
	* x: an int used to generate the key(ignore this for now)

-----

## 11/27/18 - piping

-----

## 11/26/18 - file redirection

**[free ssh](cloud9.io)**

* you can type ```cat``` and start typing(without a source file), it will output the stdin as stdout
* you can use ```>>``` to output twice, or ```>>>``` to output thrice, etc...
* ```>``` is overwrite,(redirects stdout) ```>>``` is append, ```2>``` redirects stderr(overwrites the file, while ```2>>``` appends)
* you need to use ```2>``` to redirect error msgs(stderror). ```&>``` will output stderror and stdout into file.
	* eg: ```javac out.txt 2> error.txt```
* you can also output to two files
	* eg: ```ls > out.txt 2> out2.txt```

**redirection(piping)**

* outputs the stdout of first argument to stdin of next thing
	* eg: ```ls -l | grep .txt | wc```
* grep is basically a search
* ```<``` redirects stdin from a file. The file is treated exactly like stdin, for example scanf() will read up until a newline is fonud
* | (pipe) redirect stdout from one program to stdin of the next. Very useful for chaining progams together

#### ```dup2``` - ```<unistd.h>```

* ```dup2(fd1, fd2)```: redirects fd2 to fd1. *any* use of fd2 will now act as the file for fd1. closes the file that fd2 had refered to.

#### ```dup``` - ```<unistd.h>```

* duplicates an existing entry in the file table. returns a new file descriptor for the duplicate entry
* duping ```dup2(7, 1) ``` replaces 1 with 7
* **eg**:
```
int fd1, backup_stdout;
backup_sdout = dup( STDOUT_FILENO )
dup2(fd1, STDOUT_FILENO )
dup2(backup_stdout, STDOUT_FILENO )
```

* a | b will run a in its entirety, output it to a temporary file, redirect it into b, and then run b, then delete the temporary file.


-----

## 11/19/18 - shell

* ```cd```, ```ls```, ```exit()``` are commands not programs in linux. Commands are located in one of the bins
* you want a child process to execute commands/programs so that it doesnt overwrite the shell
* **```&= 127```** makes any number positive(bit-wise operators)

## 11/16/18 - wait

* wait will wait until child process stops running
* ```waitpid(pid, status, options)```:
	* **pid**: the pid of the specific child to look for. if -1, will wait for any child(normal wait)
	* **options**: can set other behavior for ```waitpid```, if 0, will work normally

-----

## 11/15/18 - forking

**```execvp()``` terminates the current program if successful, which is why you need a ```strerror()``` to print out errnos if ```execvp()``` doesnt work**

#### ```fork()``` - ```<unistd.h>```
* creates a separate process based on the current one, the new process is called a child, the original is the parent
* the child process is a duplicate of the parent process
* all parts of the parent process are copied, including stack and heap memory, and the file table
* returns 0 to the child and the child's PID, or -1(errno) to the parent
* this allows your main file to run while a child of the program runs ```execvp()```

#### Managing wub processes: ```wait``` - ```<sys/wait.h>```
* stops a parent process from running until any child has exited
* returns the pid of the child that exited, or -1(errno), and gathers information about the child process(this is called reaping)
* if multiple child processes exit, an arbitrary one will be reaped
* ```wait(status)```
	* ```status``` is used to store information about how the process exited
	* **```status```** macros:
		* **usage**: ```MACRO(status)```
			* ```WIFEEXITED```: true if child exited normally
			* ```WEXISTATUS```: the return value of the child
			* ```WIFSIGNALED```: true if child exited due to a signal
			* ```WTERMSIG```: the signal number intercepted by the child

```
fork();
fork();
printf("Split!\n"); // this will print "split!" in terminal twice
```

-----

## 11/14/18 - work day

**```char **``` is a pointer to a pointer(So you can pass it in a function and modify it)**

-----

## 11/13/18 - executing
#### The ```exec``` family - ```<unistd.h>```
* a group of c functions that can be used to run other programs.
* replaces the curent process with the new program(turns the other program into this program)

#### execl
* ```execl(path, command, arg0, arg1 ... NULL)```
	* **path**: the path to the program(eg: "/bin/ls")
	* **command**: the name of the program(eg: "ls")
	* **arg0**... : each command line argument you wish to give the program(eg: "-a," "-l")

#### execlp
* ```execlp(path, command, arg0, arg1, ... NULL)```
	* works like ```execl```, except it uses the $PATH environment variable commands
	* for example, you can us "ls" as the *path* instead of "/bin/ls"
	* to check the path, use: ```echo $PATH```

#### execvp
* ```execvp(path, argument_array)```
	* **argument_array**: array of strings containing the arguments to the command
	* argument_array[0] must be the name of the program
	* the last entry **must** be NULL

#### strsep- ```<string.h>```

parse a string with a common delimiter
* ```strsep(source, delimiter)```
 	* locates the first occurence of any of the spiecified *delimiters* in a string and replaces it with NULL
	* *delimiters* is a string, each character is interpreted as a distinct delimiter
	* returns the beginning of the original string, sets *source* to the string starting at 1 index past the location of the new NULL
	* since *source*'s value is changed, it must be a pointer to a string(char **).

example:
```
char line[100] = "woah-this-is-cool";
char *s1 = line;
strsep(&s1, "-")
```

* replaces the - after wow with NULL
* returns a pointer to the w in "woah"
* sets s1 to point to the t in "this-is-cool"
* **we can use ```strsep``` to pass paths into ```execvp```**

-----

## 11/08/18 - signals

- you can man 7 signals to read more about signals
- ```kill -9``` sends kill signal
- seg faults are sent by OS to terminate code(```kill - 11``` gives seg fault errors)
- ```kill -SIGSTOP``` PID stops a program, and ```fg``` can resume it
- after killing a process, it may still be in the foreground, so you can type(fg) to get rid of them
- ```kill -L``` lists all killable processes
- ```ps -au``` shows all processes
- use ```| grep PID``` to get **just** that process
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
	while(1) {
		printf("hello, I'm: %d\n", getpid() );
		sleep(1);
	}
}

```

Signals in C programs: ```<signal.h>```

**kill**

- ```kill(pid, signal); ```
- returns(errno) 0 on success or -1 if not

**signal**

- ```signal(SIGNUM, sighandler) ``` this attaches a signal to the sighandler function(it "listens" for each signal)
- ```getpid();``` gets the pid of running process
- ```SIGKILL``` cannot be intercepted
- **you** can decide what to do after intercepting a signal, you can kill it with ```kill -SIGUSR1```
- you dont want to ```kill -9``` your web server/bash etc because there are some things it must do before it closes, and it cant do so if you force it to close.
- ```kill -L``` lists all processes you can kill
* files under /proc lists all files such as uptime

-----

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
- getppid() - ```<unistd.h>```
  - returns the current process's parent pid
  - What happens when you close a parent process?
    - subprocesses DO NOT ALWAYS close
    - an orphaned child's ppid will be set to 1 to show parent is not there
    - orphaned processes dont get "Adopted"
  - you can kill a process with its PID
  - $kill -9 bash: closes current bash and all children
  - $kill -9 -1: logs you out

-----

# 11/07/18 -- Processes

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
* ```getpid()``` - ```<unistd.h>```
	* returns the current process's pid
    * ```getppid()``` - ```<unistd.h>```
        * returns the current process's parent pid
* What happens when you close a parent process?
    * subprocesses **DO NOT ALWAYS** close
	* an orphaned childs ppid will be set to 1 to show parent is not there
        * orphaned processes dont get "Adopted"
    * you can kill a process with its PID
    * ```kill -9 bash```: closes current bash and all children
    * ```kill -9 -1```: logs you out
