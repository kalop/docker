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
* 1.Security
    * 1.1.Seccomp profiles
    * 1.2.Linux Kernel Capabilities and Docker
* 2.Networking
* 3.Orchestration

#### 1. Security
#### 1.1.Seccomp profiles
https://training.play-with-docker.com/security-seccomp/
 Seccomp is a sandboxing in Linux kernel that acts like a firewall for system calls. It uses Berkeley Packet Filter (BPF) rules to filter syscalls and control how they are handled. These filters can significantly limit a containers access to the Docker Host’s Linux kernel - especially for simple containers/applications.

 ##### 1.1.1 Clone the labs Github repo
 ```
$ git clone https://github.com/docker/labs
$ cd labs/security/seccomp
 ```
