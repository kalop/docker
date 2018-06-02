# Play with Docker Classroom
https://training.play-with-docker.com/
## Getting Started Walk-through for IT Pros and System Administrators

### Stage 2 - Digging Deeper
https://training.play-with-docker.com/ops-stage2/

#### What will we learn?
* Understand the architecture of Docker, and the core features
* Understand how to integrate Docker into your existing application infrastructure
* Develop a proof of concept application deployment

#### Content:
* [1.Security](#1.Security)
    * [1.1.Seccomp profiles](####1.1.Seccomp-profiles)
    * [1.2.Linux Kernel Capabilities and Docker](####1.2.Linux-Kernel-Capabilities-and-Docker)
* 2.Networking
* 3.Orchestration

#### 1.Security
#### 1.1.Seccomp profiles
https://training.play-with-docker.com/security-seccomp/
 Seccomp is a sandboxing in Linux kernel that acts like a firewall for system calls. It uses Berkeley Packet Filter (BPF) rules to filter syscalls and control how they are handled. These filters can significantly limit a containers access to the Docker Host’s Linux kernel - especially for simple containers/applications.

How to check if seccomp is enabled
```
$ docker info | grep seccomp
   Security Options: apparmor seccomp   

   $ grep SECCOMP /boot/config-$(uname -r)
      CONFIG_HAVE_ARCH_SECCOMP_FILTER=y
      CONFIG_SECCOMP_FILTER=y
      CONFIG_SECCOMP=y

```

 Command to start an interactive container based off the Alpine image and starts a shell process. It also applies the seccomp profile described by <profile>.json
```
$ docker run -it --rm --security-opt seccomp=<profile>.json alpine sh ...

```

##### 1.1.1 Clone the labs Github repo
 ```
$ git clone https://github.com/docker/labs
$ cd labs/security/seccomp
 ```

##### 1.1.2 Test a seccomp profile
 The fie *deny.json* seccomp profile  has an empty **syscalls** whitelist. It menas that **all** syscalls are blocked.

##### 1.1.3 Run a container with no seccomp profile
 No add a seccomp profile:
 ```
 $ docker run --rm -it --security-opt seccomp=unconfined debian:jessie sh

 ```
 Use the **strace** program to list the syscalls made by a particular run of a program. Ex: whoami
 ```
 $ strace -c -f -S name whoami 2>&1 1>/dev/null | tail -n +3 | head -n -2 | awk '{print $(NF)}'

 or

 strace -c -f -S name ls 2>&1 1>/dev/null | tail -n +3 | head -n -2 | awk '{print $(NF)}'

 # more verbose:
 $ strace whoami
 ```

##### 1.1.4 Selectively remove syscalls
* **deny.json** has no syscalls permitted
* **default.json** has all syscalls
* **default-no-chmod.json** has no chmod syscalls

It's a way to put security in your containers

##### 1.1.5 Write a seccomp profile
Possible actions for a syscall:

* **SCMP_ACT_TRAP**	Send a SIGSYS signal without executing the system call
* **SCMP_ACT_ERRNO** Set errno without executing the system call
* **SCMP_ACT_TRACE** Invoke a ptracer to make a decision or set errno to -ENOSYS
* **SCMP_ACT_ALLOW** Allow

The most important actions for Docker users are **SCMP_ACT_ERRNO** and **SCMP_ACT_ALLOW**.

![alt text](https://github.com/kalop/docker/blob/master/doc/img/layout_docker_seccomp.png "Layout seccomp")

More granular filters based on the value of the arguments to the system call:

* **index** is the index of the system call argument
* **op** is the operation to perform on the argument. It can be one of:
    * **SCMP_CMP_NE** - not equal
    * **SCMP_CMP_LT** - less than
    * **SCMP_CMP_LE** - less than or equal to
    * **SCMP_CMP_EQ** - equal to
    * **SCMP_CMP_GE** - greater than
    * **SCMP_CMP_GT** - greater or equal to
    * **SCMP_CMP_MASKED_EQ** - masked equal: true if (value & arg == valueTwo)
* **value** is a parameter for the operation
* **valueTwo** is used only for SCMP_CMP_MASKED_EQ

![alt text](https://github.com/kalop/docker/blob/master/doc/img/granular_filters.png "granular filter")

The rule only matches if **all** args match. Use **strace** to get all system calls to writte policies.

##### 1.1.6 A few gotchas

* **Timing of a seccomp profile application**: Polices tend to be applied very early in the container creation process. Then it needs to add syscalls **not** required for your container.
    * to avoid: use the *--security-opt no-new-privileges* flag when starting your container
* **Seccomp escapes**: *ptrace* is disabled by default because it allows bypassing of seccomp. Use [this script](https://gist.github.com/thejh/8346f47e359adecd1d53) to test for seccomp escapes through *ptrace*  
* **Using multiple filters**: The only way to use multiple seccomp filters, as of Docker 1.12, is to load additional filters within your program at runtime. [man page](http://man7.org/linux/man-pages/man2/seccomp.2.html)
* **Misc**: Reports errors without crashing the program: One such way is to use SCMP_ACT_TRAP and write your code to handle SIGSYS and report the errors in a useful way. [How Firefox handles seccomp violations](https://wiki.mozilla.org/Security/Sandbox/Seccomp)

**Further reading**:
* Very comprehensive presentation about seccomp that goes into more detail than this document.
    * Article: https://lwn.net/Articles/656307/
    * Slides: http://man7.org/conf/lpc2015/limiting_kernel_attack_surface_with_seccomp-LPC_2015-Kerrisk.pdf
* Chrome’s DSL for generating seccomp BPF programs: https://cs.chromium.org/chromium/src/sandbox/linux/bpf_dsl/bpf_dsl.h?sq=package:chromium&dr=CSs


#### 1.2.Linux Kernel Capabilities and Docker
