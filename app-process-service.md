# Application, Process, Service File, Service User — Relation


•  You start a web server software program, which creates a process in your operating system memory.
•  That program is designed to run quietly in the background as a system service.
•  The service tells the operating system to "bind" or listen to a specific port (such as port 80 for HTTP traffic).
•  When an incoming data packet arrives at your computer's IP address on port 80, the operating system uses that port number to deliver the data directly to the correct process

Concept	What it means for mysqld	How you interact with it
As a Process	It is the actual executable running in the system memory. The d stands for daemon, which means it runs quietly in the background handling database requests.	Checked via: ps aux | grep mysqld or top
As a Service	It is a configuration wrapper that allows the operating system to automatically start, stop, monitor, and manage the mysqld process.	Managed via: sudo systemctl start mysql





## 1. Application

Software/program stored on disk.

Examples:

- NGINX
- MySQL

Example file:

```text
/usr/sbin/nginx
```

---

## 2. Process

When an application runs:

- OS loads it into RAM
- OS assigns a PID
- Running instance becomes a process

Example:

```text
PID 14050 nginx
```

The process does the actual work.

---

## 3. Service Definition File (`.service`)

A configuration file used by `systemd` to manage an application/process.

Example:

```text
nginx.service
```

It can contain:

```text
ExecStart=/usr/sbin/nginx
```

It tells `systemd` how to start, stop, restart, and manage Nginx.

---

## 4. systemd

Linux service manager.

Responsibilities:

- Starts services
- Stops services
- Restarts services
- Monitors services/processes

Example:

```bash
systemctl start nginx
```

`systemctl` sends the command to `systemd`.

---

## 5. Service User / Non-Interactive User

A special Linux user used to run services with limited privileges.

Examples:

```text
nginx
mysql
postgres
```

These users are usually not intended for interactive login.

Example shell:

```text
/sbin/nologin
```

The service process runs under this user.

Example:

```text
Nginx worker process → nginx user
```

---

# How Everything Is Connected

When you install Nginx:

```bash
dnf install nginx
```

The Nginx package typically installs:

- Nginx application files
- `nginx.service`
- Nginx service user/group (depending on the package)

Flow:

```text
dnf install nginx
        ↓
Nginx package installed
        ↓
nginx.service file + nginx service user created
        ↓
systemd reads nginx.service
        ↓
systemctl start nginx
        ↓
Nginx service starts
        ↓
Nginx process runs
        ↓
PID assigned
        ↓
Runs in background
        ↓
Listens on port 80
```

---

# Important Distinction

```text
nginx
    ↓
Application/program

nginx.service
    ↓
Service definition file
    ↓
Instructions for systemd

Nginx service
    ↓
Managed background service

Nginx process
    ↓
Running instance of Nginx
    ↓
Has a PID

nginx user
    ↓
User account used to run Nginx processes
```

## 🧠 Easy Memory

**Package → Application → Service File → systemd → Service → Process → PID → Service User**

> **Service file = instructions**  
> **Service = managed background application**  
> **Process = running instance of the application**