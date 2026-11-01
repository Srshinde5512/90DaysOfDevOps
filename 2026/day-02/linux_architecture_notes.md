# Linux Under the Hood: A DevOps Quick Reference

## 1. Core Architecture: The Three Layers

* **Hardware**: CPU, RAM, disk, network controllers.
* **Kernel Space**:
  * The core software managing hardware resources.
  * Direct access to system memory and CPU rings (Ring 0).
  * Exposes interfaces to user space via **system calls (syscalls)** like `fork()`, `read()`, and `write()`.
* **User Space**:
  * Where applications, services, and shell scripts execute.
  * Restricted permissions (Ring 3); cannot touch hardware directly.
  * Isolated per process to prevent one crashed app from bringing down the entire OS.

---

## 2. Process Creation & Lifecycle

Every running application is a process with a unique **PID** (Process ID).

### How Processes Are Born
1. **`fork()`**: A parent process clones itself, creating a child process with an identical copy of memory and environment.
2. **`exec()` / `execve()`**: The child process replaces its memory space with a new executable binary.
3. **`wait()`**: The parent waits for the child to finish and reaps its exit status.

### Process States
* **R (Running / Runnable)**: Actively executing on CPU or waiting in the scheduler run queue.
* **S (Interruptible Sleep)**: Waiting for an event or I/O (e.g., keystroke, network packet, disk read); wakes up on signal.
* **D (Uninterruptible Sleep)**: Waiting directly on device hardware I/O; **cannot** be killed by signals (even `kill -9`).
* **T (Stopped)**: Suspended via user intervention (e.g., `Ctrl+Z`, `SIGSTOP`).
* **Z (Zombie / Defunct)**: Process completed execution, but its parent hasn't called `wait()` to clear its entry from the process table. *(Takes up no memory/CPU, only a PID).*

---

## 3. Init System & systemd

* **PID 1**: The first process executed by the kernel at the end of the boot sequence. All other system processes descend from it.
* **Why systemd matters**:
  * **Parallel boot**: Spawns services concurrently using socket activation, speeding up boot time.
  * **Service lifecycle**: Automatically restarts failing daemons, handles dependency graphs (starts database before app), and sets resource limits via cgroups.
  * **Unified logging**: Replaces disparate syslog text files with a structured, queryable binary log engine (`journald`).
  * **Orphan adoption**: Reaps zombie processes when their parent terminates unexpectedly.

---

## 4. Top 5 Daily Troubleshooting Commands

1. `ps aux --forest`  
   View all running processes in a hierarchical parent-child tree view.
2. `top` / `htop`  
   Real-time system monitor for per-core CPU, RAM usage, load averages, and active process states.
3. `systemctl status <service>`  
   Inspect unit runtime state, startup errors, PID, and recent log snippets.
4. `journalctl -u <service> -f`  
   Stream live application/service logs directly from the systemd journal.
5. `kill -15 <PID>` *(or `kill -9 <PID>`)*  
   Send `SIGTERM` (clean, graceful shutdown) or fallback to `SIGKILL` (immediate kernel termination).