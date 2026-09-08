# Linux Process Management

## Process Management
**Process Management:** Managing and controlling running processes.

## PID
**PID:** Unique ID assigned to a running process.

## PPID
**PPID:** PID of the parent process that started the process.

## Root Process
**Root Process:** A process running with `root` user privileges.

## PID 0
**PID 0:** Special system process; parent of kernel/system processes.

## ps
**`ps`:** Process Status — displays information about running processes.

```bash
ps

ps -ef

-e: All processes
-f: Full-format details

ps -ef
Check Process Running or Not
ps -ef | grep nginx

Only grep nginx appears → nginx is not running.

pgrep -a nginx

Output → running | No output → not running

Foreground Process

Foreground: Process occupies the terminal.

Background Process

Background: Process runs without occupying the terminal.

command &

& → Run process in background

Stop Process
Ctrl+C

Stops foreground process.

kill PID

Gracefully stops process.

kill -9 PID

Forcefully stops process.

Memory: kill PID → graceful | kill -9 PID → force

High CPU

Reasons: Heavy computation, infinite loop, too many processes, high traffic, application bug.

High Memory

Reasons: Memory leak, large application/data, too many processes, high traffic, insufficient RAM.

High CPU/Memory Troubleshooting
Check → Identify → Check Logs → Find Root Cause → Fix → Restart → Verify → Monitor
top
free -h
ps -eo pid,ppid,%cpu,%mem,cmd --sort=-%cpu | head
ps -eo pid,ppid,%cpu,%mem,cmd --sort=-%mem | head
Logs
journalctl

System/service logs

/var/log/

Traditional log files

tail -f /var/log/messages

View logs continuously

dmesg

Kernel logs

Bring Application Up
systemctl status <service>
systemctl restart <service>
systemctl status <service>

Verify:

curl http://localhost:<port>
netstat -nltp

Shows listening TCP ports and the process/PID using each port.

netstat -nltp
-n → Numeric
-l → Listening
-t → TCP
-p → Process/PID

Memory: netstat -nltp → Which process is listening on which port?