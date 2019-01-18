# Methods To Check Number of CPU Cores on Linux
Source: https://www.2daygeek.com/command-check-find-number-of-cpu-cores-linux/#

Objective:
- To understand about CPU, Cores and Threads information

## Terminology:
- **CPU Socket**: CPU socket or CPU slot is the connector on the motherboard that allows a computer processor to be connected to a motherborad. It's called physical CPU (central processing unit).
- **CPU Core**: Originally, CPU had a single core but manufactures added aditional cores in to that to increase performance, that's why core came to picture. For example, a dual-core CPU has two central processing units, so it appears to the operating system as two CPUs. Like that, a quad-core CPU has four central processing units and an octa-core CPU has eight central processing units.
- **CPU Thread**: Intel Hyper-Threading Technology uses processor resources more efficiently by enabling multiple threads to run on each core (each core can run two threads). It also increases processor throughput and improving overall performance on threaded software.

### Example
```bash
CPU(s):               32
On-line CPU(s) list:  0-31
Thread(s) per core:   2
Core(s) per socket:   8
Socket(s):            2
```
Two Physical CPU(2), Each CPU had (8) cores, and Per Core had (2) Threads.
Also, CPUs = Threads per core x Cores per socket x Sockets = 2 x 8 x 2 = 32

## 14 Methods to Check number of CPU Cores on Linux

### 1. Using /proc/cpuinfo file
/proc/cpuinfo is a virtual text file that contains information about the CPUs (central processing units) on a computer. We can gep a number of CPUs by grepping processor parameter.

```bash
grep -c ^processor /proc/cpuinfo
```

### 2. Using nproc Command
nproc - print the number of processing units available to the current process. It's part of GNU Coreutils.

```bash
nproc
```

### 3. Using lscpu Command
lscpu - display information on CPU architecture and gathers CPU architecture information like number of CPUs, threads, cores, sockets, NUMA nodes, information about CPU caches, CPU family, model and prints it in a human-readable format.

```bash
lscpu
```

Alternatively, use the lscpu command to print only the number of proccessor.

```bash
lscpu | grep 'CPU(s):' | head -1 | awk '{print $2}'
```

### 4. Using getconf Command
getconf stands for get configuration values. getconf utility used to write the value of the variable specified by system_var & path_var operand. The value of each configuration variable were obtained from IEEE Std 1003.1-2001

```bash
getconf _NPROCESSORS_ONLN
```

### 5. Using dmidecode command
Dmidecode is a tool which reads a computer (stands for Desktop Management Interface) (some say SMBIOS - stands for System Management BIOS) table contents and display system hardware information in a human-readable format.

Suggested reading: https://www.2daygeek.com/dmidecode-get-print-display-check-linux-system-hardware-information/

This table contains a description of the system’s hardware components, as well as other useful information such as serial number, Manufacturer information, Release Date, and BIOS revision, etc,.,

```bash
-- Micron don't have dmidecode installed
dmidecode -t processor | egrep 'Designation|Count'
```

### 6. Using inxi command
inxi is a nifty tool to check hardware information on Linux and offers wide range of option to get all the hardware information on lInux system that I never found in any other utility which are available in Linux. It was forked from the ancient and mindbendingly perverse yet ingenious infobash, by locsmif.

```bash
-- Micron don't have inxi installed
inxi -C
```

### 7. Using hwinfo command
hwinfo stands for hardware information tool is another great utility that used to probe for the hardware present in the system and display detailed information about varies hardware components in human readable format.

It reports information about CPU, RAM, keyboard, mouse, graphics card, sound, storage, network interface, disk, partition, bios, and bridge, etc.

```bash
hwinfo --short --cpu
```

### 8. Using Top command
The top command provides a dynamic real-time view of a running syustem process as well as a list of task currently being managed by the Linux kernel. By default the top command shows combined of all cpu's, if you want to print all the CPU's output in the top command press 1 "Numeric One" while running top utility.

Suggested Reading: https://www.2daygeek.com/top-command-examples-to-monitor-server-performance/

```bash
top -- click 1 to print all the CPU's output
```

### 9. Using htop command

Htop is an open-source ncurses-based process viewer for Linux. It is similar to top, but allows you to scrool vertically and horizontally, so ou can see all the process running on the system, along with their full command lines. Tasks related to processes (killing, renicing) can be done without entering their PIDs.
--htop are not installed in Micron

### 10. Using CorFreq Tool
CoreFreq is a CPU monitoring software designed for Intel 64-bits Processors and supported architectures are Atom, Core2, Nehalem, SandyBridge and superior, AMD Family 0F. CoreFreq provides a framework to retrieve CPU data with a high degree of precision. By default CoreFreq output displays number of CUP cores in the system.
--CoreFreq are not installed in Micron

### 11. Using /sys/devices/system/cpu/file
Use the /sys/devices/system/cpu/ interface to get a processor information. It's a collection of both global and individual CPU attributes. Individual CPU attributes are contained in sub-directories named by the kernel's logical CPU number.
- online: cpus that are online are being scheduled.
- possible: cpus that have been allocated resources and can be brought online if they are present.
- present: cpus that have been indetified as being present in the system

```bash
cat /sys/devices/system/cpu/online
cat /sys/devices/system/cpu/possible
cat /sys/devices/system/cpu/present

echo $((`cat /sys/devices/system/cpu/present | sed 's/0-//'` + 1))
echo $((`cat /sys/devices/system/cpu/online | sed 's/0-//'` + 1))
echo $((`cat /sys/devices/system/cpu/possible | sed 's/0-//'` + 1))
```
### 12. Using /proc/interrupts file
The file /proc/interrupts contains information about the interrupts, like how many times processor has been interrupted.
```bash
cat /proc/interrupts | egrep -i 'cpu'
```

### 13. Using Python multiprocessing.cpu_count() method
You can count the number of cores on the system using python multiprocessing.cpu_count() method. Alternatively import os module using syscall getconf.

```bash
python -c 'import multiprocessing as mp; print mp.cpu_count()'
python -c 'import multiprocessing as mp; print(mp.cpu_count())'

python -c 'import os; print os.sysconf(os.sysconf_names["SC_NPROCESSORS_ONLN"]);'
```
### 14. Using Python psutil library
psutil (Python system and process utilities is a cross-platform library for retrieving information on running process and system utilization (CPU, memory, disks, network, sensors) in Python.

```python
>>> import psutil
>>> psutil.cpu_count()
```
