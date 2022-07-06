Authors:

**_Roland Ziyi Guo@Tencent Xuanwu Lab_   
_Yi He@Network and Information Security Lab, Tsinghua University_**

**ATTENTION:** this document is not an official paper, we write this for illustrating the essential model for providers, users. Also, apply for CVEs.
# fundamentals
For nowadays design of ebpf in linux kernel,  we can easily load an ebpf program with CAP_SYS_ADMIN or CAP_BPF, the behaviors of the ebpf programs in kernel are somehow cannot be strictly limited(we call this, "ambiguous security behaviors boundary"), especailly for the type of trace point, kprobe/kretprobe ebpf. Great penetrating power of ebpf in kernel could result in many harmful outcomes in cloud world, even if that was not its intention.
# methods
For raw_trace_point, which was introduced to linux since v4.17, attckers can easily load malicious ebpf program to kernel in a container with SYS_ADMIN, catch and modify almost every process in VM globally, no matter where the process is(i.e. a process inside the container or a process in host VM).
Specifically, we have implemented many different versions(not only  raw_trace_point) evil ebpf programs for lots of processes, the result shows its potential threats. For example, the system calls' sequences of process  ```cron```('PATH: "/usr/sbin/cron" in most linux") have been catched by us. After hijacking its behaviors,   we inejct the payload to its user buffer and execute the payload as full root user outside the container. What has to be mentioned is that we not only develop the evil ebpf program for cron(cron is JUST ONE OF THEM), but also for many other processes, they have different attacking paths based-on this model. Besides, we have also implemented the evil ebpf programs for ```python```, ```shell```, ```docker & dockerd```, ```sshd```, ```kubelet```, etc...
# scenarios
The details of our further analysis, methods, codes, offense framework, threat model, and defense or mitigations will be public after our paper is formally published. Because some details of this part could bring potential harmful for Providers, so we totally understand the security and ethics concerns.
# temporary mitigation
1. For some containers, using seccomp to ban or filter sys_bpf().
2. Drop SYS_ADMIN or CAP_BPF, sometimes it is hard, see: https://github.com/docker/docker.github.io/issues/13731 , https://github.com/moby/moby/issues/43374
3. turn off bpf syscall in Kernel
   ```
   CONFIG_BPF=n
   CONFIG_BPF_SYSCALL=n
   ```
# new design
Furthermore, it doesn't only seem to be just a careless or simple security problem, because it is quite inappropriate that banning ebpf becomes the only ways(for Now) to mitigate. As nowadays development of Multi-tenant cloud service, this type of threat would be more and more universal. We do believe that linux kernel has some problems about the design of "security behaviors boundary" and "privilege granularity" in some types of ebpf programs. Perhaps we need better permission model for ebpf or some specific behavior boundary, and we are still fighting for this.
# acknowledgement
https://defcon.org/html/defcon-29/dc-29-speakers.html#path
