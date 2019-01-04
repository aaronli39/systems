# SYSTEMS - Aaron Li

## 01/04/19 - networking continued

**socket**
* a connection between 2 programs over a *network*
* a socket corresponds to an IP address
* a computer has 65,000 connections available
* example ipv6: 0000 : 0000 : 0000 : 0000 : 004f : 13c2 : 0009 : a2d2
* ipv4 to ipv6: 149.89.150.100 -> :: ffff : 149.89.150.100
* if ISPs are smart, they can just give every home their own octet at the end of the ipv6 address, which would also lower latency
* there are **2^128** ipv6 adddreses

**to use a socket**(similar to named pipes):
1) create socket
2) bind it to an address/port
3) listen/initiate a conversation
4) send/receive data
   
* most services assume default port(port 80), and if you type another number like port 81 it will fail to connect because it's not a known service. 
* games dont use well known ports such as port 22(ssh) because then they wouldn't be able to ssh since the ssh server won't be active
* the port is kind of like the well known pipe, once something connects to the port, a process can open another secret connection in order to communicate.

-----

## 01/03/19 - networking

* router splits signals and it knows who each signal is for
* stuy has 65k addresses for the CS department(just the cs department).
* IPv4 has 4 octets
* IPv6 has hexadecimals, and some blocks were backwards compatible
* 127 block of IPv4 is "loopback" and every home network has one. However, you need a router to connect to the rest of the internet so that it knows where to route internet traffic to. 

-----

## 12/19/18 - nothing

-----

## 12/18/18 - handshake agreement

**handshake**:
* a procedure to ensure that a connection has been established between 2 programs
* both ends of the connection must verify that they can send and receive data to and from each other
* 3 way handshake(so that server/client knows they are engaged in "conversation"):
	* client sends a message to the server
	* server sends a response to client
	* client sneds response back to server
* how can we set up this connection?:
	* client create a named pipe, and tell the server the name of that pipe. now the server can open that pipe and send a response. it will write to the pipe and the client will send a respnose back(3rd way) to make sure its working.

**server client/design pattern**:
* 1 - setup:
	* 1 - server creates a well known pipe FIFO and waits for a connection(cus everyone shud know about it)
	* 2 - client creates a "private" FIFO
* 2 - handshake:
	* 1 - client connects to server and sends the private FIFO name. client waits for a response frmo the server
	* 2 - server receives client's message and removes the WKP(well known pipe, or the server's pipe)
	* 3 - server connects to client FIFO, sending an initial acknowledgement message
	* 4 - client receives server's message and removes its private FIFO
	* 5 - client sends response to server
* **NOTE:** this handshake happens automatically when using network stuff(TCPIP)
* 3 - operation
	* 1 - server and client send information back and forth(compilaton setp)
* 4 - reset:
	* 1 - client exits, server closes any connection to the client
	* 2 - server recreates the WKP and waits for another client
-----

## 12/17/18 - name pipes(FIFOs)

* named pipes are pipes that exist outsside a process/program
* "p" for pipe in permissions in the termainal with ```ll```
* it uses no space/disk until it is being used/called
* ```cat > warrio``` will allow u to write to wario and you can view it with ```cat warrior``` on output
* dont open a pipe twice from ```readin```(the one catting warrio)
* if no stdout for warrio exists(closing a connection to one end of the pipe), then the stdin will close(the one you're typing ```cat > warrio``` into)
* warrio will be size 0 even if you catted stuff in(always 0) it is never on disk and always buffered memory
* you can pipe warrio into wc for example, and it will work as intended(```cat > warrio``` and in another terminal u have ```cat warrio | wc```)
* if you ```rm warrio``` it still works, you can access it outside a program
* ```mkfifo```:  shell command to make a FIFO. (```$ mkfifo name```)

**```mkfifo - <sys/types.h> <sys/stat.h>```**
* c function to create a FIFO
* returns 0 on success and -1 on failure
* once created, the FIFO will act like a regular file, and we can use open, read, write, and close on it
* FIFOs willl block on open until both ends of the pipe have a connect(as soon as someone closes their side of the pipe, the whole thing is gone)
* ```mkfifo( name, permissions)```

```c
// create pip, open, and print

int main() {
	int fd;

	mkfifo("luigi", 0644);
	print("pipe created\n");
	fd = open("luigi", O_RDWR);
	printf("pipe opened: %s\n", fd);

	return 0;
}
```
* you can use semophores to control who can write to a FIFO
* parallel project idea wud be cool
* do not **NEED** a client

-----

## 12/14/18 - hyperthreading/multithreading

* 8 threads available: 4 cores available and 4 leftovers
* these "extra" cores give a boost to some processes soemtimes by doing a bit of extra work

-----

## 12/10/18 - controlling resources with semophores

* IPC construct used to control acess to a shared resources(like a file or shared memory)
* most commonly, a semophore is used as a counter representing how many processes can access a resource at a given time
	* if a semophore has a value of 3, then it can have 3 active "users"

#### Semophore opeartions:

* create a semophore
* set initial avalue
* up(s) / v(s) - atomic, if you up a semophore, u have to use down later. down = s--, up = s++. **upping** a semophoer unblocks a process.
* `down(s) / P(s) `
  * attempt to take the semophore
  * if the semophore is 0, wait fr it to be available
  * 0 means you dont have access, 1+ means more than 1 user

* eg: dowwnning s means it wont be available till done
```c
down(s);
open(f);
read/write
up(s)
```

#### semget

* get/access a semophore
* returns the semophore decriptor to -1
* ```semget(key, const, flags)```
	* key: unique identifier
	* amount: semophores are stored as sets of one or more. The number of semophores to create/get in the set
	* flags: includes permissions for the semophore
		* IPC_CREAT

#### semctl - <sys/types.h> <sys/ipc.h> <sys/semh>
* control the semophore, including:
  * set the semophore value
  * remove the semophore
  * get the current value
  * get/set semohphore metadata


* **semctl(descriptor, index, operation, data):**
  * descriptor: the retun value of semget
  * index: the index of the semophore you want to control in the semophore set
  * operation: **you only need to use one of these**
    * IPC_RMID: remove the semaphore
    * SETVAL: set the value(requiers data)
  	* GETVAL: return the value
* data: variable for setting/storing semophore metadata
* type union semun: **you have to declare this union in your main c file:**

```c
union semun {
	int val; // used for SETVAL
	struct semid_ds *buf; // Used for IPC_STAT and IPC_SET
	unsigned short *array; // used for SETALL
	struct seminfo *__buff;
};
```

####  semop()

* perform an atomic sequence
* you can up/down a semaphore by any integer value, not just 1
* `semop( descriptor, operation, amount)`
  * amount: the amt of operations you want to perform on semaphore set
  * peration: a pointer to a struct sembuf:
```c
struct sembuf {
	short sem_op;
	short sem_num;
	short sem_flag;
}
```
* sem num: the index of teh semaphore you want to work on
* sem op: down(s): any negative number, up(s): any positive number, 0: block until semaphore reaches 0
* sem flag:
* sem_UNDO: allow the os to undo the given operation, useful in the event a program exsts before it could release
-----

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
