---
layout: post
title: A Very Hypertextual Introduction to my Project
tags: [io_uring, qemu]
author: Aarushi Mehta
redirect_from: "/2019/06/15/hypertext-intro-to-my-project"
---
How would I explain my project to a beginner? By adding so very many details I would like to share in hyperlinks. My project is officially described [here](https://wiki.qemu.org/Internships/ProjectIdeas/IOUring). I'd like this to help in getting started with understanding how QEMU and async I/O & resources to gain more understanding.

### What and Why.
QEMU is software that lets you --

    1. Run a 'guest' operating system on your host operating system - emulating a full system or a Virtual Machine
    2. Run programs compiled for one computer architecture  (say, ARM Cortex A8) on another (x86_64)  - known as user mode emulation. 

There are a wide variety of uses for this. My first ever use case for QEMU was trying to write programs that would run on the [Raspberry Pi](https://github.com/wimvanderbauwhede/limited-systems/wiki/Raspbian-%22stretch%22-for-Raspberry-Pi-3-on-QEMU) before I had one. Or it could be playing Windows only [games on Linux](https://davidyat.es/2016/09/08/gpu-passthrough/). QEMU is also popular in [Operating Systems development](http://blog.vmsplice.net/2011/02/near-instant-kernel-development-cycle.html) because it solves the problem of how to test your new OS code without leaving your own machine unbootable or surviving slow reboot cycles. There are so many options.

Doing all the above things well requires QEMU to be fast. Trying to run a virtual machine has overhead obviously, you run a program on the guest that is converted into code that can run on your host, so it'll be slower than any program that just runs on the host. The experience of performance is relative -- if something ran twice as slow on the guest OS as your host you would notice. Recently, Solid State Disks have made Input/Output on storage much faster. The kind of overhead that was acceptable in the past feels much worse.

![Virtualisation I/O Overhead](/assets/block-latency.png){:class="img-responsive"}
From Stefan Hajnoczi's [talk](https://vmsplice.net/~stefan/stefanha-kvm-forum-2017.pdf) at KVM Forum

Asynchronous or non squential I/O does not wait for requests to be completed. Once a request is submitted, there is no blocking of the CPU while a disk request is being completed. The CPU can move to other tasks. In light of that, there are usually two approaches, the disk interrupts the CPU when requests are actually completed or the application polls for completion. Polling is just checking repeatedly whether completion has been achieved -- like an annoying repetition of *Are We Done Yet?* till the disk says *Yes we are*. This is valuable in use cases where most of the time is spent waiting on Disk I/O because that is much slower than the speed of CPU execution. For example, during a disk operation that takes ten milliseconds to perform, a processor that is clocked at one gigahertz could have performed ten million instruction-processing cycles. 

> If you're curious about how I/O tends to work on Linux -- a free chapter of TLPI is [here](https://nostarch.com/download/TLPI_Ch4.pdf) after which I would recommend [this](https://eklitzke.org/blocking-io-nonblocking-io-and-epoll)

[IO_Uring](http://kernel.dk/io_uring.pdf) (<- this is some excellent documentation of io_uring) is a brand new asynchronous I/O interface that aims to supplant the older [Linux AIO](https://www.fsl.cs.sunysb.edu/~vass/linux-aio.txt) interface by being faster and cooler and more efficient. QEMU can benefit from being able to use it.

From my original internship description, IO_Uring offers the following optimisations:
1. A single system call can both submit and complete I/O requests or polling mode can be used to avoid system calls altogether
    Every system call in QEMU is expensive because it requires the guest to wait for the host to perform it and then kick it back to the guest. IO_uring automatically starts completing any submissions it receives. It also saves memory, because the ring stucture holding request status and data is shared between the kernel and the userspace. Polling mode just lets you directly add events to the ring and automatically submits and completes the requests without a system call.

2. Memory buffers can be registered (pinned) ahead of time to avoid pinning on each request
    Memory buffers indicate where data is stored temporarily. Doing I/O on it requires us to map these buffers to the main memory, doing so repeatedlyis expensive, hence, the ability to use the same set of buffers throughout the lifecycle of a ring.

3. File descriptors can be held long-term to avoid the need to acquire them on each request
    The kernel acquires file descriptors and drops them at the end of each request. Doing this repeatedly for different requests costs a lot of time especially because file descriptors are atomic in nature. This means that either instructions succeed in their entirety, uninterrupted, or fail to execute at all. This time can be saved if a set of commonly used file descriptors are just saved.


QEMU is a large and complex project that is not completely documented, but [this](https://www.qemu.org/2018/02/09/understanding-qemu-devices/) explains how QEMU works and of course the resources on the [QEMU wiki](https://wiki.qemu.org/Documentation) are good places to start before you jump into code.

A good way of understanding how code works is [tracing](https://jvns.ca/blog/2016/09/17/strange-loop-talk/) the calls it makes, QEMU tracing [documentation is here](https://git.qemu.org/?p=qemu.git;a=blob_plain;f=docs/devel/tracing.txt;hb=HEAD)


### The How
QEMU I/O is implemented at a block level. Block devices are hardware devices distinguished by the random (not necessarily sequential) access of fixed-size chunks of data or 'blocks'. Usually, you emulate a `virtio-blk` or `virtio-scsi` device at your qemu system. There is support for two asynchronous IO engines -- one using thread pools and one using Linux AIO. 

First, I write a basic 'engine' file that calls io_uring system calls and does processing.
It's called from the block I/O code in `block/fileposix.c`, so I enable the engine to be called from there
I add the ability to use io_uring by adding options to read it from the command line by adding it to the list of possible options in the generator.
Then, I test and benchmark it.
Add the advanced features of io_uring.
Test and Benchmark :)