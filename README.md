<div align="center">

# PerfSpect
Analyze and Optimize Linux Servers

[![Build](https://github.com/intel/PerfSpect/actions/workflows/build-test.yml/badge.svg)](https://github.com/intel/PerfSpect/actions/workflows/build-test.yml)[![CodeQL](https://github.com/intel/PerfSpect/actions/workflows/codeql.yml/badge.svg)](https://github.com/intel/PerfSpect/actions/workflows/codeql.yml)[![License](https://img.shields.io/badge/License-BSD--3-blue)](https://github.com/intel/PerfSpect/blob/master/LICENSE)

[Getting PerfSpect](#getting-perfspect) | [Running PerfSpect](#running-perfspect) | [Building PerfSpect](#building-perfspect-from-source)
</div>

## What is PerfSpect
This command-line tool is designed to help you analyze and optimize Linux servers and the software running on them. Whether you’re a system administrator, a developer, or a performance engineer, this tool provides comprehensive insights and actionable recommendations to enhance performance and efficiency.

## Getting PerfSpect
```
$ wget -qO- https://github.com/intel/PerfSpect/releases/latest/download/perfspect.tgz | tar xvz
$ cd perfspect
```
## Running PerfSpect
PerfSpect includes a suite of commands designed to analyze and optimize both system and software performance.

Run `perfspect -h` for top level help. Note the available commands and options.

### Quick Start for Users of Intel PerfSpect 1.x
PerfSpect has undergone a re-design for the 3.x release. Prior releases of PerfSpect required two stages: 

1) collect raw event data
2) process the raw event data into user-friendly metrics

The 3.x design combines these two steps into one...producing user-friendly metrics. Run `perfspect metrics` for the new experience.

Also available with 3.x are "live metrics" where metrics are printed to stdout as they are collected. Try `perfspect metrics --live`.

### Quick Start for Users of Intel System Health Inspector (AKA "svr-info") 2.x
Svr-info functionality is now included in PerfSpect. The svr-info configuration report is generated by running `perfspect report`. Benchmarks can be executed by running `perfspect report --benchmark all`. Run `perfspect flame` to generate a software flamegraph. And, run `perfspect telemetry` to collect system telemetry.

### Commands
| Command | Description |
| ------- | ----------- |
| [`perfspect config`](#config-command) | Modify system configuration |
| [`perfspect flame`](#flame-command) | Generate flamegraphs |
| [`perfspect lock`](#lock-command) | Collect system wide hotspot, c2c and lock contention information |
| [`perfspect metrics`](#metrics-command) | Monitor core and uncore metrics |
| [`perfspect report`](#report-command) | Generate configuration report |
| [`perfspect telemetry`](#telemetry-command) | Collect system telemetry |

Each command has additional help text that can be viewed by running `perfspect <command> -h`.

#### Config Command
The `config` command provides a method to view and change various system configuration parameters. Run `perfspect config -h` to view the parameters that can be modified. <b>USE CAUTION</b> when changing system parameters. It is possible to configure the system in a way that it will no longer operate. In some cases, a reboot will be required to return to the default settings. 

Example:
```
$ ./perfspect config --cores 24 --llc 2.0 --uncoremaxfreq 1.8
...
```
#### Flame Command
Software flamegraphs are useful in diagnosing software performance bottlenecks. Run `perfspect flame` to capture a system-wide software flamegraph. **Important Note:** The target system must have perl installed and on the PATH to process the data required for flamegraphs.

#### Lock Command
As systems contain more and more cores, it can be useful to analyze the kernel lock overhead and potential false-sharing that impacts system scalability. Run `perfspect lock` to collect system wide hotspot, c2c and lock contention information. Experienced performance engineers can analyze the collected information to identify bottlenecks.

#### Metrics Command
The `metrics` command provides system performance characterization metrics. The metrics provided are dependent on the platform architecture.

Example:
```
$ ./perfspect metrics --duration 30
emr                   ⣯  collection complete                     

Metric files:
  /home/jharper5/dev/pt/perfspect_2024-10-10_10-58-36/emr_metrics.csv
  /home/jharper5/dev/pt/perfspect_2024-10-10_10-58-36/emr_metrics_summary.csv
  /home/jharper5/dev/pt/perfspect_2024-10-10_10-58-36/emr_metrics_summary.html
```
The `metrics` command supports two modes -- default and "live". Default mode behaves as above -- metrics are collected and saved into files for review.  The "live" mode prints the metrics in a selected format, e.g., CSV, JSON, to stdout where they can be viewed in the console and/or redirected into a file or observability pipeline.

##### No Root Permissions
If sudo is not possible and running as the root user is not possible, use the `--noroot` flag on the command line, e.g., `perfspect metrics --noroot`, and request an administrator make the following changes to the target system:
- sysctl -w kernel.perf_event_paranoid=0
- sysctl -w kernel.nmi_watchdog=0
- write '125' to all perf_event_mux_interval_ms files found under /sys/devices/*, e.g., `for i in $(find /sys/devices -name perf_event_mux_interval_ms); do echo 125 > $i; done`

See `perfspect metrics -h` for the extensive set of options and examples.

#### Report Command
The `report` command generates system configuration reports in a variety of formats. By default, all categories of information are collected. See `perfspect report -h` for all options.
```
$ ./perfspect report 
soc-PF4W5A3V          ⢿  collection complete                     

Report files:
  /home/myuser/dev/perfspect/perfspect_2024-09-03_17-45-40/soc-PF4W5A3V.html
  /home/myuser/dev/perfspect/perfspect_2024-09-03_17-45-40/soc-PF4W5A3V.xlsx
  /home/myuser/dev/perfspect/perfspect_2024-09-03_17-45-40/soc-PF4W5A3V.json
  /home/myuser/dev/perfspect/perfspect_2024-09-03_17-45-40/soc-PF4W5A3V.txt
```
It's possible to collect a subset of information by providing command line options. Note that by specifying only the `txt` format, it is printed to stdout, as well as written to a report file.
```
$ ./perfspect report --bios --os --format txt
BIOS
====
Vendor:       Intel Corporation
Version:      EGSDCRB1.SYS.1752.P05.2401050248
Release Date: 01/05/2024

Operating System
================
OS:              Ubuntu 23.10
Kernel:          6.5.0-44-generic
Boot Parameters: BOOT_IMAGE=/boot/vmlinuz-6.5.0-44-generic root=UUID=e6d667af-f0b7-450b-b409-9fe2647aeb38 ro
Microcode:       0x21000230

Report files:
  /home/myuser/dev/perfspect/perfspect_2024-09-03_17-47-55/emr.txt
```
###### Benchmarks
To assist in evaluating the health of target systems, the report command can run a series of micro-benchmarks. The results will be reported along with the target's configuration details. Use --benchmarks \<"all" or comma separated list of benchmarks\>:
```
$ ./perfspect report --benchmarks all
soc-PF4W5A3V          ⢿  collection complete                     

Report files:
  /home/myuser/dev/perfspect/perfspect_2024-09-03_17-45-40/soc-PF4W5A3V.html
  /home/myuser/dev/perfspect/perfspect_2024-09-03_17-45-40/soc-PF4W5A3V.xlsx
  /home/myuser/dev/perfspect/perfspect_2024-09-03_17-45-40/soc-PF4W5A3V.json
  /home/myuser/dev/perfspect/perfspect_2024-09-03_17-45-40/soc-PF4W5A3V.txt
```

| benchmark | Description |
| --------- | ----------- |
| all | runs all benchmarks |
| speed | runs each [stress-ng](https://github.com/ColinIanKing/stress-ng) cpu-method for 1s each, reports the geo-metric mean of all results. |
| power | runs stress-ng in two stages: 1) load 1 cpu to 100% for 20s to measure maximum frequency, 2) load all cpus to 100% for 60s. Uses [turbostat](https://github.com/torvalds/linux/tree/master/tools/power/x86/turbostat) to measure power. |
| temperature | runs the same micro benchmark as 'power', but extracts maximum temperature from turbostat output. |
| frequency | runs [avx-turbo](https://github.com/travisdowns/avx-turbo) to measure scalar and AVX frequencies across processor's cores. **Note:** Runtime increases with core count.  |
| memory | runs [Intel(r) Memory Latency Checker](https://www.intel.com/content/www/us/en/download/736633/intel-memory-latency-checker-intel-mlc.html) (MLC) to measure memory bandwidth and latency across a load range. **Important Note:** MLC is not included with PerfSpect. It can be downloaded from here: [MLC](https://www.intel.com/content/www/us/en/download/736633/intel-memory-latency-checker-intel-mlc.html). Once downloaded, extract the Linux executable and place it in the perfspect/tools/x86_64 directory. |
| numa | runs Intel(r) Memory Latency Checker(MLC) to measure bandwidth between NUMA nodes. See Note above about downloading MLC. |
| storage | runs [fio](https://github.com/axboe/fio) for 2m in read/write mode with a single worker to measure single-thread read and write bandwidth. Use the --storage-dir flag to override the default location. Minimum 5GB disk space required to run test. |

#### Telemetry Command
The `telemetry` command runs telemetry collectors on the specified target(s) and then generates reports of the results. By default, all telemetry types are collected. To select telemetry types, additional command line options are available (see `perfspect telemetry -h`).
```
$ ./perfspect telemetry --duration 30
soc-PF4W5A3V          ⣾  collection complete                     

Report files:
  /home/myuser/dev/perfspect/perfspect_2024-09-03_17-55-13/soc-PF4W5A3V_telem.html
  /home/myuser/dev/perfspect/perfspect_2024-09-03_17-55-13/soc-PF4W5A3V_telem.xlsx
  /home/myuser/dev/perfspect/perfspect_2024-09-03_17-55-13/soc-PF4W5A3V_telem.json
  /home/myuser/dev/perfspect/perfspect_2024-09-03_17-55-13/soc-PF4W5A3V_telem.txt
```

### Common Command Options

#### Local vs. Remote Targets
By default, PerfSpect targets the local host, i.e., the host where PerfSpect is running. Remote system(s) can also be targetted when the remote systems are reachable through SSH from the local host.

**Important:** Ensure the remote user has password-less sudo access (or root privileges) to fully utilize PerfSpect's capabilities.

To target a single remote system using a pre-configured private key:
```
$ ./perfspect report --target 192.168.1.42 --user fred --key ~/.ssh/fredkey
...
```
To target a single remote system using a password:
```
$ ./perfspect report --target 192.168.1.42 --user fred
fred@192.168.1.42's password: ******
...
```
To target more than one remote system, a YAML file is used to provide the necessary connection parameters, e.g.:
```
$ cat targets.yaml
# This YAML file contains a list of remote targets with their corresponding properties.
# Each target has the following properties:
#   name: The name of the target (optional)
#   host: The IP address or host name of the target (required)
#   port: The port number used to connect to the target via SSH (optional)
#   user: The user name used to connect to the target via SSH (optional)
#   key: The path to the private key file used to connect to the target via SSH (optional)
#   pwd: The password used to connect to the target via SSH (optional)
#
# Note: If key and pwd are both provided, the key will be used for authentication.
#
# Security Notes: 
#   It is recommended to use a private key for authentication instead of a password.
#   Keep this file in a secure location and do not expose it to unauthorized users.
#
# Below are examples. Modify them to match your environment.
targets:
  - name: ELAINES_TARGET
    host: 192.168.1.1
    port: 
    user: elaine
    key: /home/elaine/.ssh/id_rsa
    pwd:
  - name: JERRYS_TARGET
    host: 192.168.1.2
    port: 2222
    user: jerry
    key:
    pwd: george

$ ./perfspect report --benchmark speed,memory --targets targets.yaml
...
```
## Building PerfSpect from Source
### 1st Build
`builder/build.sh` builds the dependencies and the app in Docker containers that provide the required build environments. Assumes you have Docker installed on your development system.

### Subsequent Builds
`make` builds the app. Assumes the dependencies have been built previously and that you have Go installed on your development system.
