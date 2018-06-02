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
* [1.Security](#1security)
    * [1.1.Seccomp profiles](#11seccomp-profiles)
    * [1.2.Linux Kernel Capabilities and Docker](#12linux-kernel-capabilities-and-docker)
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

##### 1.2.1.Introduction to capabilities
The Linux kernel is able to break down the privileges of the root user into distinct units referred to as **capabilities**. Almost all of the special powers associated with the Linux root user are broken down into individual capabilities and allows you to:
* Remove individual capabilities from the root user account, making it less powerful/dangerous.
* Add privileges to non-root users at a very granular level.

##### 1.2.2.Working with Docker and capabilities
* Drop capabilities from the root:

    ```
    $ docker run --rm -it --cap-drop $CAP alpine sh

    ```
* Add capabilities to the root:
code
    ```
    $ docker run --rm -it --cap-add $CAP alpine sh

    ```
* Drop all capabilities and then add individual ones

    ```
    $ docker run --rm -it --cap-drop ALL --cap-add $CAP alpine sh
    ```
[Capabilities man page](http://man7.org/linux/man-pages/man7/capabilities.7.html)


##### 1.2.3.Testing Docker Capabilities

* Start a new container and prove that the container’s root account can change the ownership of files.

    ```
    $  docker run --rm -it alpine chown nobody /

    ```
    * The command gives no return code indicating that the operation succeeded. The command works because the default behavior is for new containers to be started with a root user. This root user has the CAP_CHOWN capability by default.


* Start another new container and drop all capabilities for the containers root account other than the CAP_CHOWN capability.

    ```
    $ docker run --rm -it --cap-drop ALL --cap-add CHOWN alpine chown nobody /
    ```
    * This command also gives no return code, indicating a successful run. The operation succeeds because although you dropped all capabilities for the container’s root account, you added the chown capability back. The chown capability is all that is needed to change the ownership of a file.


* Start another new container and drop only the CHOWN capability form its root account
```
$  docker run --rm -it --cap-drop CHOWN alpine chown nobody /
```
    * This time the command returns an error code indicating it failed. This is because the container’s root account does not have the CHOWN capability and therefore cannot change the ownership of a file or directory.


* Create another new container and try adding the CHOWN capability to the non-root user called nobody
    ```
    $  docker run --rm -it --cap-add chown -u nobody alpine chown nobody /
    ```
    * The command fails because Docker does not yet support adding capabilities to non-root users.


##### 1.2.4.Extra for experts

* **libcap**
    * **capsh** - lets you perform capability testing and limited debugging
    * **setcap** - set capability bits on a file
    * **getcap** - get the capability bits from a file
* **libcap-ng**
    * **pscap** - list the capabilities of running processes
    * **filecap** - list the capabilities of files
    * **captest** - test capabilities as well as list capabilities for current process


* The following command will start a new container using Alpine Linux, install the libcap package and then list capabilities.

```
$ docker run --rm -it alpine sh -c 'apk add -U libcap; capsh --print'
```
* **Current** is multiple sets separated by spaces. Multiple capabilities within the same set are separated by commas ,. The letters following the + at the end of each set are as follows:
    * **e** is effective
    * **i** is inheritable
    * **p** is permitted


* The capsh command can be useful for experimenting with capabilities. capsh --help shows how to use the command:
```
$ docker run --rm -it alpine sh -c 'apk add -U libcap;capsh --help'
```

* Modify Capabilities
    *  set the CAP_NET_RAW capability as effective and permitted
    ```
    $ setcap cap_net_raw=ep $file  --> setcap command calls on libcap to do this.
    ```
    * Use libcap-ng to set the capabilities of a file
    ```
    $ filecap /absolute/path net_raw  --> The filecap command calls on libcap-ng.
    ```

* Auditing:
    * libcap:
        ```
        $ getcap $file

        $ $file = cap_net_raw+ep

        ```
    * libcap-ng:

    ```
    $ filecap /absolue/path/to/file
    file                     capabilities
/absolute/path/to/file        net_raw
    ```
    * extended atributes    

    ```
    $ getfattr -n security.capability $file
    # file: $file
    security.capability=0sAQAAAgAgAAAAAAAAAAAAAAAAAAA=
    ```
* Tips: it is possible to mount volumes that contain files with capability bits set into containers
    * audit directories for capability bits
    ```
    # with libcap
    getcap -r /

    # with libcap-ng
    filecap -a
    ```
    * remove capability bits
    ```
    # with libcap
    setcap -r $file

    # with libcap-ng
    filecap /path/to/file none
    ```

[Futher reading](https://www.kernel.org/doc/ols/2008/ols2008v1-pages-163-172.pdf)
