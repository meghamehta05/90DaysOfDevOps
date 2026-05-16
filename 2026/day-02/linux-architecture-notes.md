# Linux Architecture, Processes, and systemd

## Linux Architecture

### Kernel
- Core part of Linux OS
- Manages CPU, memory, devices, and processes
- Acts as bridge between hardware and software

### User Space
- Area where users interact with applications
- Includes shell, commands, and software programs

### Init / systemd
- First process started during boot
- systemd manages services and background processes

---

## Process Management

A process is a running program in Linux.

### Process States
- Running → currently executing
- Sleeping → waiting for resource/input
- Stopped → paused process
- Zombie → completed but still exists in process table

### Important Process Commands

```bash
ls
cd
pwd
top
cd dir